---
title: 'python 内核分析（六）：List类型对象'
date: 2018-10-08 17:12:03
tags: PYTHON
categories: PYTHON # 分类
thumbnail: /assets/img/posts/PYTHON-KERNEL/default.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# 关于List对象

我们首先去`[Include/listobject.h]`中去寻找列表对象的定义:

```c
[Include/listobject.h]

typedef struct {
    PyObject_VAR_HEAD
    /* Vector of pointers to list elements.  list[0] is ob_item[0], etc. */
    PyObject **ob_item;

    /* ob_item contains space for 'allocated' elements.  The number
     * currently in use is ob_size.
     * Invariants:
     *     0 <= ob_size <= allocated
     *     len(list) == ob_size
     *     ob_item == NULL implies ob_size == allocated == 0
     * list.sort() temporarily sets allocated to -1 to detect mutations.
     *
     * Items must normally not be NULL, except during construction when
     * the list is not yet visible outside the function that builds it.
     */
    Py_ssize_t allocated;
} PyListObject;
```

其中有三个属性，`PyObject_VAR_HEAD`是指向了一个list元素,以及一个二级指针`**ob_item`储存了这个指针的地址。还有一个`allocated`储存分配的内存大小。

```c
/* PyObject_VAR_HEAD defines the initial segment of all variable-size
 * container objects.  These end with a declaration of an array with 1
 * element, but enough space is malloc'ed so that the array actually
 * has room for ob_size elements.  Note that ob_size is an element count,
 * not necessarily a byte count.
 */
#define PyObject_VAR_HEAD      PyVarObject ob_base;
```

我们可以看见`PyObject_VAR_HEAD`指向了一个`PyVarObject`，即其指向了这个元素的第一个，我们通俗地看就是`list[0]`就是`ob_item[0]`。

我们看指向的`PyVarObject`，其中的属性`ob_size`就代表了当前列表的个数`len(list)`。

我们要注意的是，其会申请一块较大的内存，及我们有`0 <= ob_size <= allocated`关系。

但当列表元素为`NULL`时，整个内存分配会直接降为`0`，此时有，`ob_size == allocated == 0`

# PyListObject 方法

我们可以看见在这个`Object`类中的方法:

```c
[Include/listobject.h]

PyAPI_DATA(PyTypeObject) PyList_Type;
PyAPI_DATA(PyTypeObject) PyListIter_Type;
PyAPI_DATA(PyTypeObject) PyListRevIter_Type;
PyAPI_DATA(PyTypeObject) PySortWrapper_Type;

#define PyList_Check(op) \
    PyType_FastSubclass(Py_TYPE(op), Py_TPFLAGS_LIST_SUBCLASS)
#define PyList_CheckExact(op) (Py_TYPE(op) == &PyList_Type)

PyAPI_FUNC(PyObject *) PyList_New(Py_ssize_t size);
PyAPI_FUNC(Py_ssize_t) PyList_Size(PyObject *);
PyAPI_FUNC(PyObject *) PyList_GetItem(PyObject *, Py_ssize_t);
PyAPI_FUNC(int) PyList_SetItem(PyObject *, Py_ssize_t, PyObject *);
PyAPI_FUNC(int) PyList_Insert(PyObject *, Py_ssize_t, PyObject *);
PyAPI_FUNC(int) PyList_Append(PyObject *, PyObject *);
PyAPI_FUNC(PyObject *) PyList_GetSlice(PyObject *, Py_ssize_t, Py_ssize_t);
PyAPI_FUNC(int) PyList_SetSlice(PyObject *, Py_ssize_t, Py_ssize_t, PyObject *);
PyAPI_FUNC(int) PyList_Sort(PyObject *);
PyAPI_FUNC(int) PyList_Reverse(PyObject *);
PyAPI_FUNC(PyObject *) PyList_AsTuple(PyObject *);
#ifndef Py_LIMITED_API
PyAPI_FUNC(PyObject *) _PyList_Extend(PyListObject *, PyObject *);

PyAPI_FUNC(int) PyList_ClearFreeList(void);
PyAPI_FUNC(void) _PyList_DebugMallocStats(FILE *out);
#endif
```

# PyListObject的创建

列表的创建我们使用的是`PyList_New()`：

```c
[Include/listobject.h]

PyObject *
PyList_New(Py_ssize_t size)
{
    PyListObject *op;
#ifdef SHOW_ALLOC_COUNT
    static int initialized = 0;
    if (!initialized) {
        Py_AtExit(show_alloc);
        initialized = 1;
    }
#endif

    if (size < 0) {
        PyErr_BadInternalCall();
        return NULL;
    }
    if (numfree) {
        numfree--;
        op = free_list[numfree];
        _Py_NewReference((PyObject *)op);
#ifdef SHOW_ALLOC_COUNT
        count_reuse++;
#endif
    } else {
        op = PyObject_GC_New(PyListObject, &PyList_Type);
        if (op == NULL)
            return NULL;
#ifdef SHOW_ALLOC_COUNT
        count_alloc++;
#endif
    }
    if (size <= 0)
        op->ob_item = NULL;
    else {
        op->ob_item = (PyObject **) PyMem_Calloc(size, sizeof(PyObject *));
        if (op->ob_item == NULL) {
            Py_DECREF(op);
            return PyErr_NoMemory();
        }
    }
    Py_SIZE(op) = size;
    op->allocated = size;
    _PyObject_GC_TRACK(op);
    return (PyObject *) op;
}
```

我们在这里看见了`numfree`这个属性，返回去看，发现其与一个列表`free_list[]`有关，称其为`freelist`，也可以叫其为缓冲池。其常量定义在最前面：

```c
[Include/listobject.h]

/* Empty list reuse scheme to save calls to malloc and free */
#ifndef PyList_MAXFREELIST
#define PyList_MAXFREELIST 80
#endif
static PyListObject *free_list[PyList_MAXFREELIST];
static int numfree = 0;
```

在缓冲池有剩余时，新建一个`list`对象，会直接从缓冲区`freelist[]`中直接初始化，缓冲池不可用的时候，就会调用`PyObject_GC_New()`直接新建。

默认情况下，缓冲区最大有`PyList_MAXFREELIST = 80`个对象。

`PyObject_GC_New()`是`_PyObject_GC_New()`的别名：

```c
[Modules/gcmodules.c]

PyObject *
_PyObject_GC_New(PyTypeObject *tp)
{
    PyObject *op = _PyObject_GC_Malloc(_PyObject_SIZE(tp));
    if (op != NULL)
        op = PyObject_INIT(op, tp);
    return op;
}
```

其为这个对象类型分配内存。

而关于缓冲区`freelist[]`的函数其实只有三个，另外俩一个是`PyList_ClearFreeList()`，用以清除所有的缓冲区；还有一个是`list_dealloc()`，其作用是`PyListType`对象的析构操作。

奇怪的是，其会在释放掉`PyListObject *op`中维护的所有对象后，视情况将其放入缓冲区或者直接释放掉。这应该是为了保证缓冲区在结束后有足够的大小。

# PyListObject的元素设置

```c
[Objects/listobject.c]

int
PyList_SetItem(PyObject *op, Py_ssize_t i,
               PyObject *newitem)
{
    PyObject **p;
    if (!PyList_Check(op)) {
        Py_XDECREF(newitem);
        PyErr_BadInternalCall();
        return -1;
    }
    if (i < 0 || i >= Py_SIZE(op)) {
        Py_XDECREF(newitem);
        PyErr_SetString(PyExc_IndexError,
                        "list assignment index out of range");
        return -1;
    }
    p = ((PyListObject *)op) -> ob_item + i;
    Py_XSETREF(*p, newitem);
    return 0;
}
```

在进行了类型和越界检查后，平移指针找到需要替换的位置，替换为新的对象。在`Py_XSERREF()`中我们能看见其将原本对象的引用进行检查，必要时施行销毁操作。
# PyListObject获取元素

```c
[Objects/listobject.c]

PyObject *
PyList_GetItem(PyObject *op, Py_ssize_t i)
{
    if (!PyList_Check(op)) {
        PyErr_BadInternalCall();
        return NULL;
    }
    if (i < 0 || i >= Py_SIZE(op)) {
        if (indexerr == NULL) {
            indexerr = PyUnicode_FromString(
                "list index out of range");
            if (indexerr == NULL)
                return NULL;
        }
        PyErr_SetObject(PyExc_IndexError, indexerr);
        return NULL;
    }
    return ((PyListObject *)op) -> ob_item[i];
}
```

# PyListObject的插入操作

```c
[Objects/listobject.c]

static int
ins1(PyListObject *self, Py_ssize_t where, PyObject *v)
{
    Py_ssize_t i, n = Py_SIZE(self);
    PyObject **items;
    if (v == NULL) {
        PyErr_BadInternalCall();
        return -1;
    }
    if (n == PY_SSIZE_T_MAX) {
        PyErr_SetString(PyExc_OverflowError,
            "cannot add more objects to list");
        return -1;
    }

    if (list_resize(self, n+1) < 0)
        return -1;

    if (where < 0) {
        where += n;
        if (where < 0)
            where = 0;
    }
    if (where > n)
        where = n;
    items = self->ob_item;
    for (i = n; --i >= where; )
        items[i+1] = items[i];
    Py_INCREF(v);
    items[where] = v;
    return 0;
}

int
PyList_Insert(PyObject *op, Py_ssize_t where, PyObject *newitem)
{
    if (!PyList_Check(op)) {
        PyErr_BadInternalCall();
        return -1;
    }
    return ins1((PyListObject *)op, where, newitem);
}
```

简单地处理了插入时的越界和负索引「如果负索引过小于`-n`会直接置零」后，确定插入位置，后面的元素统一后移一个位置。

# PyListObject的附加操作

```c
[Objects/listobject.c]

static int
app1(PyListObject *self, PyObject *v)
{
    Py_ssize_t n = PyList_GET_SIZE(self);

    assert (v != NULL);
    if (n == PY_SSIZE_T_MAX) {
        PyErr_SetString(PyExc_OverflowError,
            "cannot add more objects to list");
        return -1;
    }

    if (list_resize(self, n+1) < 0)
        return -1;

    Py_INCREF(v);
    PyList_SET_ITEM(self, n, v);
    return 0;
}

int
PyList_Append(PyObject *op, PyObject *newitem)
{
    if (PyList_Check(op) && (newitem != NULL))
        return app1((PyListObject *)op, newitem);
    PyErr_BadInternalCall();
    return -1;
}
```

和上面的添加操作差不多

# PyListObject的删除操作

```c
[Objects/listobject.c]

static PyObject *
list_remove(PyListObject *self, PyObject *value)
/*[clinic end generated code: output=f087e1951a5e30d1 input=2dc2ba5bb2fb1f82]*/
{
    Py_ssize_t i;

    for (i = 0; i < Py_SIZE(self); i++) {
        int cmp = PyObject_RichCompareBool(self->ob_item[i], value, Py_EQ);
        if (cmp > 0) {
            if (list_ass_slice(self, i, i+1,
                               (PyObject *)NULL) == 0)
                Py_RETURN_NONE;
            return NULL;
        }
        else if (cmp < 0)
            return NULL;
    }
    PyErr_SetString(PyExc_ValueError, "list.remove(x): x not in list");
    return NULL;
}
```

python会在遍历列表后删除找到第一个匹配的元素，并在找不到时返回异常。

# 其他操作

此外PyListObject还有很多方法，比如`merge`相关，`reverse`相关和`sort`相关等等，这些在以后再写。
