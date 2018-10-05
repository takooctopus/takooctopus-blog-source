---
title: 'python 内核分析（二）：PyObject，对象的基础'
date: 2018-09-25 19:08:52
tags: PYTHON
categories: PYTHON # 分类
thumbnail: /assets/img/posts/PYTHON-KERNEL/1.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# PyObject，对象的基础

## PyObject的定义
我们就用官方里面的解释：
没有什么能被完全被声明为一个`PyObject`，但每一个指向`Python`的对象的指针均能够被视为一个`PyObject*` 指针。
```language-c
  [Include/object.h]

  /* Nothing is actually declared to be a PyObject, but every pointer to
  * a Python object can be cast to a PyObject*.  This is inheritance built
  * by hand.  Similarly every pointer to a variable-size Python object can,
  * in addition, be cast to PyVarObject*.
  */
  typedef struct _object {
    _PyObject_HEAD_EXTRA
    Py_ssize_t ob_refcnt;
    struct _typeobject *ob_type;
  } PyObject;
```
### _PyObject_HEAD_EXTRA

再接着我们看里面的第一个属性`_PyObject_HEAD_EXTRA`，追踪到
其也定义在object.h里面：

```language-c
  [Include/object.h]

  #ifdef Py_TRACE_REFS
  /* Define pointers to support a doubly-linked list of all live heap objects. */
  #define _PyObject_HEAD_EXTRA            \
    struct _object *_ob_next;           \
    struct _object *_ob_prev;

  #define _PyObject_EXTRA_INIT 0, 0,

  #else
  #define _PyObject_HEAD_EXTRA
  #define _PyObject_EXTRA_INIT
  #endif
```
其表示我们如果定义了`Py_TRACE_REFS`的话，这个结构将可以支持双向列表，并且所有堆中的活动均在这个列表中
>因为在 **release** 下没有定义`Py_TRACE_REFS`，我们可以忽略这个宏

### Py_ssize_t

接着我们追踪`Py_ssize_t`这个属性

```language-c
[Include/pyport.h]

/* Py_ssize_t is a signed integral type such that sizeof(Py_ssize_t) ==
 * sizeof(size_t).  C99 doesn't define such a thing directly (size_t is an
 * unsigned integral type).  See PEP 353 for details.
 */
#ifdef HAVE_SSIZE_T
typedef ssize_t         Py_ssize_t;
#elif SIZEOF_VOID_P == SIZEOF_SIZE_T
typedef Py_intptr_t     Py_ssize_t;
#else
#   error "Python needs a typedef for Py_ssize_t in pyport.h."
#endif

```
关于`HAVE_SSIZE_T`
查到对于`HAVE_SSIZE_T`的默认定义为1，其定义在 **pyconfig.h** 中

关于`ssize_t`
即我们默认下，根据平台将`ssize_t`的类型设置为`__int64`或者`_W64`
```language-c
  /* Define like size_t, omitting the "unsigned" */
  #ifdef MS_WIN64
  typedef __int64 ssize_t;
  #else
  typedef _W64 int ssize_t;
  #endif
  #define HAVE_SSIZE_T 1
```

关于`Py_intptr_t`
我们继续看`Py_intptr_t`其实是由`intptr_t`定义的
```language-c
  typedef intptr_t        Py_intptr_t;
```

我们继续看`intptr_t`
其又在 **Include/vcstdint.h** 中被定义了：

```language-c
  // 7.18.1.4 Integer types capable of holding object pointers
  #ifdef _WIN64 // [
    typedef __int64           intptr_t;
    typedef unsigned __int64  uintptr_t;
  #else // _WIN64 ][
    typedef _W64 int               intptr_t;
    typedef _W64 unsigned int      uintptr_t;
  #endif // _WIN64 ]
```
我们看到其在64位系统中是64位的int，而32位系统是_W64，我们可以看到`Py_ssize_t`的本质为一个int

### 此时，我们就可以简化我们所看见的对象了

```language-c
  typedef struct _object {
    int ob_refcnt;
    struct _typeobject *ob_type;
} PyObject;
```

成员作用:
- `ob_refcnt`：counter，对于变量引用次数进行计数，关于垃圾清理。
- `ob_type`：type，储存了对象的类型信息，如`int`,`string`,或是`function`等

除此之外，这个作为一个基本单元，在实际使用时还需要其他的内容

我们以 **PyLongObject** 为例：

```language-c
[Included/longobject->Inlcude/longintrepr.h]

  struct _longobject {
    PyObject_VAR_HEAD
    digit ob_digit[1];
};
```
其第一个属性`PyObject_VAR_HEAD`由`PyVarObject`和`ob_base`组成
第二个属性由`digit`构成

## 变长对象和定长对象
在c语言中, 整型占用内存大小是固定的, 无论你保存1还是100. 而由于c没有字符串类型, 因此要表示字符串得用N个char组成数组. 在python中这样描述变长对象:

```language-c
typedef struct {
    PyObject ob_base;
    Py_ssize_t ob_size; /* Number of items in variable part */
} PyVarObject;
```
`ob_size` 表示容纳元素的个数, 不是对象占用的内存字节数.

| int           | string        | list     | dict      |
| ------------- |:-------------:| --------:|----------:|
| ob_refnt      | ob_refnt      | ob_refnt | ob_refnt  |
| ob_type       | ob_type       | ob_type  | [others]  |
| [NULL]        | [others]      | [others] | [NULL]    |

## 类型对象

```language-c
  [Include/object.h]
  
  #ifdef Py_LIMITED_API
  typedef struct _typeobject PyTypeObject; /* opaque */
  #else
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
  #endif
```

在 `_typeobject` 中包含了很多信息,主要有:

- 1.类型名，`tp_name`，格式为 **&lt;module&gt;.&lt;name&gt;** ，用于内部调用和方便调试
- 2.创建对象时分配内存空间大小的变量，`tp_basicsize`和`tp_itemsize`，分别代表基本的大小和里面元素的大小
- 3.与对象相关的操作，如`tp_print`这样的函数
- 4.要描述的本类型的其他信息