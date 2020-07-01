---
title: 'python 内核分析（三）：对象的创建'
date: 2018-09-26 23:20:07
tags: 
- CS
- PYTHON
categories: CS # 分类
thumbnail: /assets/img/posts/CS/PythonKernel.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# 对象的创建

Python在关于对象的创建上，提供了两种方式：
- 1.C Api
- 2.通过类型创建，比如`PyLong_Type`「Objects/longobject.c」

而在`C Api`中提供了两类：
- 1.一种是泛型的Api，形式如同`PyObject_Xxx`「Include/object.h」，其能够应用在任何Python对象上。
- 2.另一种是与数据类型相关的Api：
```c
    PyAPI_FUNC(PyObject *) PyLong_FromLong(long);
```

不管使用哪种模块，最终都是直接分配相应的内存。

## 对象的行为

我们继续去看`PyTypeObject`的定义，在「Include/object.h」中我们能够看见一个关于`PyTypeObject`的定义结构体：

```c
[Include/object.h]

typedef struct _typeobject {
    PyObject_VAR_HEAD
    const char *tp_name; /* For printing, in format "<module>.<name>" */
    Py_ssize_t tp_basicsize, tp_itemsize; /* For allocation */

    /* Methods to implement standard operations */

    destructor tp_dealloc;
    printfunc tp_print;
    getattrfunc tp_getattr;
    setattrfunc tp_setattr;
    PyAsyncMethods *tp_as_async; /* formerly known as tp_compare (Python 2)
                                    or tp_reserved (Python 3) */
    reprfunc tp_repr;

    /* Method suites for standard classes */

    PyNumberMethods *tp_as_number;
    PySequenceMethods *tp_as_sequence;
    PyMappingMethods *tp_as_mapping;

    /* More standard operations (here for binary compatibility) */

    hashfunc tp_hash;
    ternaryfunc tp_call;
    reprfunc tp_str;
    getattrofunc tp_getattro;
    setattrofunc tp_setattro;

    /* Functions to access object as input/output buffer */
    PyBufferProcs *tp_as_buffer;

    /* Flags to define presence of optional/expanded features */
    unsigned long tp_flags;

    const char *tp_doc; /* Documentation string */

    /* Assigned meaning in release 2.0 */
    /* call function for all accessible objects */
    traverseproc tp_traverse;

    /* delete references to contained objects */
    inquiry tp_clear;

    /* Assigned meaning in release 2.1 */
    /* rich comparisons */
    richcmpfunc tp_richcompare;

    /* weak reference enabler */
    Py_ssize_t tp_weaklistoffset;

    /* Iterators */
    getiterfunc tp_iter;
    iternextfunc tp_iternext;

    /* Attribute descriptor and subclassing stuff */
    struct PyMethodDef *tp_methods;
    struct PyMemberDef *tp_members;
    struct PyGetSetDef *tp_getset;
    struct _typeobject *tp_base;
    PyObject *tp_dict;
    descrgetfunc tp_descr_get;
    descrsetfunc tp_descr_set;
    Py_ssize_t tp_dictoffset;
    initproc tp_init;
    allocfunc tp_alloc;
    newfunc tp_new;
    freefunc tp_free; /* Low-level free-memory routine */
    inquiry tp_is_gc; /* For PyObject_IS_GC */
    PyObject *tp_bases;
    PyObject *tp_mro; /* method resolution order */
    PyObject *tp_cache;
    PyObject *tp_subclasses;
    PyObject *tp_weaklist;
    destructor tp_del;

    /* Type attribute cache version tag. Added in version 2.6 */
    unsigned int tp_version_tag;

    destructor tp_finalize;

#ifdef COUNT_ALLOCS
    /* these must be last and never explicitly initialized */
    Py_ssize_t tp_allocs;
    Py_ssize_t tp_frees;
    Py_ssize_t tp_maxalloc;
    struct _typeobject *tp_prev;
    struct _typeobject *tp_next;
#endif
} PyTypeObject;
```

在这整个结构体中，定义了许多函数指针，其指向了某个函数或者`NULL`，这些函数指针定义了Python类型对象的操作，其会在对象运行的状态而表现出不同的行为。

我们以上面11行的代码为例：

```c
[Include/object.h]

 printfunc tp_print;
```

我们继续追踪，能在前面发现定义:

```c
[Include/object.h]

typedef int (*printfunc)(PyObject *, FILE *, int);
```

明显的，这个属性最终给出了一个函数指针。

## 对象流程

明显的`tp_new`与`tp_init`定义了对象的创建于初始化操作。

而对于标准类有几个重要的方法：`tp_as_number`、`tp_as_sequence`和`tp_as_mapping`

>我们以第一个属性为例，追踪找到其表示的类`PyNumberMethods`：
```c
[Include/object.h]

typedef struct {
    /* Number implementations must check *both*
       arguments for proper type and implement the necessary conversions
       in the slot functions themselves. */

    binaryfunc nb_add;
    binaryfunc nb_subtract;
    binaryfunc nb_multiply;
    binaryfunc nb_remainder;
    binaryfunc nb_divmod;
    ternaryfunc nb_power;
    unaryfunc nb_negative;
    unaryfunc nb_positive;
    unaryfunc nb_absolute;
    inquiry nb_bool;
    unaryfunc nb_invert;
    binaryfunc nb_lshift;
    binaryfunc nb_rshift;
    binaryfunc nb_and;
    binaryfunc nb_xor;
    binaryfunc nb_or;
    unaryfunc nb_int;
    void *nb_reserved;  /* the slot formerly known as nb_long */
    unaryfunc nb_float;

    binaryfunc nb_inplace_add;
    binaryfunc nb_inplace_subtract;
    binaryfunc nb_inplace_multiply;
    binaryfunc nb_inplace_remainder;
    ternaryfunc nb_inplace_power;
    binaryfunc nb_inplace_lshift;
    binaryfunc nb_inplace_rshift;
    binaryfunc nb_inplace_and;
    binaryfunc nb_inplace_xor;
    binaryfunc nb_inplace_or;

    binaryfunc nb_floor_divide;
    binaryfunc nb_true_divide;
    binaryfunc nb_inplace_floor_divide;
    binaryfunc nb_inplace_true_divide;

    unaryfunc nb_index;

    binaryfunc nb_matrix_multiply;
    binaryfunc nb_inplace_matrix_multiply;
} PyNumberMethods;
```

这一个结构体定义了当类型对象「PyTypeObject」被作为一个数值对象时所能够做的操作。

明显的`nb_add`代表了数值加法操作。

>顺便的，我们可以得到另外两种类型的主要类型
- `tp_as_sequence`对应`list`
- `tp_as_mapping`对应`dict`

## 类型对象的类型

我们可以看见类型对象「PyTypeObject」的第一个属性就是`PyObject_VAR_HEAD`，这代表了这个Python对象的类型同时也是一个对象。

对其进行追踪后，我们发现其在「Include/object.h」中最开始定义了这个宏：

```c
[Include/object.h]

#define PyObject_VAR_HEAD      PyVarObject ob_base;
```

分别对应了一个可变对象以及基本对象「在上一篇中我们提到了基本对象」

而类型对象的类型就是它自己。

## 对象的多态性

对于一个Python对象来说，特别的对于Python基础对象「_object」「PyObject」来说，其有一个`ob_type`属性，其中包括了Python对象的类型信息。

这里的`ob_type`就是一个`PyTypeObject`即我们上面讨论的类型对象。

我们的Python对象能够直接使用箭头调用即`object->ob_type->`形式继续调用里面的属性

而函数之间一般通过泛型指针`PyObject*`来传递

我们回头看其中的某个属性，就拿`C API`中的所列出来的一样：

```c
[Include/object.h -> Object/object.c]

Py_hash_t
PyObject_Hash(PyObject *v)
{
    PyTypeObject *tp = Py_TYPE(v);
    if (tp->tp_hash != NULL)
        return (*tp->tp_hash)(v);
    /* To keep to the general practice that inheriting
     * solely from object in C code should work without
     * an explicit call to PyType_Ready, we implicitly call
     * PyType_Ready here and then check the tp_hash slot again
     */
    if (tp->tp_dict == NULL) {
        if (PyType_Ready(tp) < 0)
            return -1;
        if (tp->tp_hash != NULL)
            return (*tp->tp_hash)(v);
    }
    /* Otherwise, the object can't be hashed */
    return PyObject_HashNotImplemented(v);
}
```

我们可以看到这个函数中进行了多态的判断。

## 对象的引用计数

在类似于C和C++以及Rust的语言中，程序员有极大的自由，能够任意的申请内存，{% post_link 2018-09-27-Program-Tips-2 其中的小问题可以看看这篇文章%}，但编程人员需要负责内存的释放问题，这其中很容易导致内存泄漏和空指针的BUG。

为了解决这些问题，现代高级语言大多采用了自己的内存管理和维护系统，像Java与C#均有其自己的垃圾回收机制。

同样的，Python也建立了其自己的垃圾回收机制。

最为明显的就是它的引用计数：

我们回去看与`ob_refcnt`的值变化有关的函数，可以在下面发现`Py_INCREF(op)`与`Py_DECREF(op)`

```c
[Include/object.h]

#define Py_INCREF(op) (                         
    _Py_INC_REFTOTAL  _Py_REF_DEBUG_COMMA       
    ((PyObject *)(op))->ob_refcnt++)

#define Py_DECREF(op)                                   
    do {                                                
        PyObject *_py_decref_tmp = (PyObject *)(op);    
        if (_Py_DEC_REFTOTAL  _Py_REF_DEBUG_COMMA       
        --(_py_decref_tmp)->ob_refcnt != 0)             
            _Py_CHECK_REFCNT(_py_decref_tmp)            
        else                                            
            _Py_Dealloc(_py_decref_tmp);                
    } while (0)
```

我们可以看见当`ob_refcnt`这个引用计数减少到0时，`Py_DECREF(op)`就会调用析构函数` _Py_Dealloc(_py_decref_tmp)`进行内存的释放工作

```c
[Include/object.h]

#define _Py_Dealloc(op) (                               
    _Py_INC_TPFREES(op) _Py_COUNT_ALLOCS_COMMA          
    (*Py_TYPE(op)->tp_dealloc)((PyObject *)(op)))
#endif
```

一般来说如果`Py_LIMITED_API`没有被定义时，`_Py_Dealloc(op)`的触发会销毁对象，释放内存。

>类型对象是超越引用计数规则的, 它永远不会被析构.并且类型对象是共享的, 多个int类型变量的类型对象指针都是指向同一个, 且不会被视为是类型对象的引用.

我们可以简单地验证一下

```python
    [test.py]

    x = 2
    y = 2
    
    x == y
    x is y
```
上面这段程序输出为`True`和`True`。

我们再看它们的`id(x)`和`id(y`)，在我的电脑上它们id均为`22941395520`，id是他们的身份唯一指定，我们也很容易看出其指向了同一个int对象。