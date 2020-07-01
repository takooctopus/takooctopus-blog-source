---
title: 'python 内核分析（十）：Python 虚拟机'
date: 2018-10-14 19:12:25
tags: 
- CS
- PYTHON
categories: CS # 分类
thumbnail: /assets/img/posts/CS/PythonKernel.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# Python虚拟机

之前生成的python字节码是无法直接运行的，这个时候我们需要使用python虚拟机，将编译得到的字节码进行执行。

而python虚拟机的实现，是以一种近似于CPU的操作方式——栈帧。

我们虚拟机其实在执行的时候，面对的是`PyFrameObject`，我们将其想象成一个空间就好

我们看其在[Include/frameobject.h]中的定义

```c

typedef struct _frame {
    PyObject_VAR_HEAD
    struct _frame *f_back;      /* previous frame, or NULL */
    PyCodeObject *f_code;       /* code segment */
    PyObject *f_builtins;       /* builtin symbol table (PyDictObject) */
    PyObject *f_globals;        /* global symbol table (PyDictObject) */
    PyObject *f_locals;         /* local symbol table (any mapping) */
    PyObject **f_valuestack;    /* points after the last local */
    /* Next free slot in f_valuestack.  Frame creation sets to f_valuestack.
       Frame evaluation usually NULLs it, but a frame that yields sets it
       to the current stack top. */
    PyObject **f_stacktop;
    PyObject *f_trace;          /* Trace function */
    char f_trace_lines;         /* Emit per-line trace events? */
    char f_trace_opcodes;       /* Emit per-opcode trace events? */

    /* Borrowed reference to a generator, or NULL */
    PyObject *f_gen;

    int f_lasti;                /* Last instruction if called */
    /* Call PyFrame_GetLineNumber() instead of reading this field
       directly.  As of 2.3 f_lineno is only valid when tracing is
       active (i.e. when f_trace is set).  At other times we use
       PyCode_Addr2Line to calculate the line from the current
       bytecode index. */
    int f_lineno;               /* Current line number */
    int f_iblock;               /* index in f_blockstack */
    char f_executing;           /* whether the frame is still executing */
    PyTryBlock f_blockstack[CO_MAXBLOCKS]; /* for try and loop blocks */
    PyObject *f_localsplus[1];  /* locals+stack, dynamically sized */
} PyFrameObject;

```

明显的，这个能够形成一个链表，通过`*f_back`能够找到之前的栈帧。在执行完当前栈帧后能够返回之前的栈帧中。

`*f_code`就是`PyCodeObject`对象

`*f_builtins`, `*f_globals` 和 `*f_locals`作为命名空间，维护了变量名和变量值之间的关系，而他们都是`PyDictObject`

`**f_valuestack`和`**f_stacktop`记录了栈底和栈顶的地址

由于每次创建栈帧对象时，`PyCodeObject`的大小都可能不同，最终其大小也可能不同，占的大小装在`PyCodeObject`中的`co_stacksize`中

***

# PyFrameObject的创建

首先，我们看栈帧的创建

```c
PyFrameObject*
PyFrame_New(PyThreadState *tstate, PyCodeObject *code,
            PyObject *globals, PyObject *locals)
{
    PyFrameObject *f = _PyFrame_New_NoTrack(tstate, code, globals, locals);
    if (f)
        _PyObject_GC_TRACK(f);
    return f;
}
```

在这之中其调用了一个私有方法`_PyFrame_New_NoTrack()`

```c
PyFrameObject* _Py_HOT_FUNCTION
_PyFrame_New_NoTrack(PyThreadState *tstate, PyCodeObject *code,
                     PyObject *globals, PyObject *locals)
{
    PyFrameObject *back = tstate->frame;
    PyFrameObject *f;
    PyObject *builtins;
    Py_ssize_t i;

#ifdef Py_DEBUG
    if (code == NULL || globals == NULL || !PyDict_Check(globals) ||
        (locals != NULL && !PyMapping_Check(locals))) {
        PyErr_BadInternalCall();
        return NULL;
    }
#endif
    if (back == NULL || back->f_globals != globals) {
        builtins = _PyDict_GetItemId(globals, &PyId___builtins__);
        if (builtins) {
            if (PyModule_Check(builtins)) {
                builtins = PyModule_GetDict(builtins);
                assert(builtins != NULL);
            }
        }
        if (builtins == NULL) {
            /* No builtins!              Make up a minimal one
               Give them 'None', at least. */
            builtins = PyDict_New();
            if (builtins == NULL ||
                PyDict_SetItemString(
                    builtins, "None", Py_None) < 0)
                return NULL;
        }
        else
            Py_INCREF(builtins);

    }
    else {
        /* If we share the globals, we share the builtins.
           Save a lookup and a call. */
        builtins = back->f_builtins;
        assert(builtins != NULL);
        Py_INCREF(builtins);
    }
    if (code->co_zombieframe != NULL) {
        f = code->co_zombieframe;
        code->co_zombieframe = NULL;
        _Py_NewReference((PyObject *)f);
        assert(f->f_code == code);
    }
    else {
        Py_ssize_t extras, ncells, nfrees;
        ncells = PyTuple_GET_SIZE(code->co_cellvars);
        nfrees = PyTuple_GET_SIZE(code->co_freevars);
        extras = code->co_stacksize + code->co_nlocals + ncells +
            nfrees;
        if (free_list == NULL) {
            f = PyObject_GC_NewVar(PyFrameObject, &PyFrame_Type,
            extras);
            if (f == NULL) {
                Py_DECREF(builtins);
                return NULL;
            }
        }
        else {
            assert(numfree > 0);
            --numfree;
            f = free_list;
            free_list = free_list->f_back;
            if (Py_SIZE(f) < extras) {
                PyFrameObject *new_f = PyObject_GC_Resize(PyFrameObject, f, extras);
                if (new_f == NULL) {
                    PyObject_GC_Del(f);
                    Py_DECREF(builtins);
                    return NULL;
                }
                f = new_f;
            }
            _Py_NewReference((PyObject *)f);
        }

        f->f_code = code;
        extras = code->co_nlocals + ncells + nfrees;
        f->f_valuestack = f->f_localsplus + extras;
        for (i=0; i<extras; i++)
            f->f_localsplus[i] = NULL;
        f->f_locals = NULL;
        f->f_trace = NULL;
    }
    f->f_stacktop = f->f_valuestack;
    f->f_builtins = builtins;
    Py_XINCREF(back);
    f->f_back = back;
    Py_INCREF(code);
    Py_INCREF(globals);
    f->f_globals = globals;
    /* Most functions have CO_NEWLOCALS and CO_OPTIMIZED set. */
    if ((code->co_flags & (CO_NEWLOCALS | CO_OPTIMIZED)) ==
        (CO_NEWLOCALS | CO_OPTIMIZED))
        ; /* f_locals = NULL; will be set by PyFrame_FastToLocals() */
    else if (code->co_flags & CO_NEWLOCALS) {
        locals = PyDict_New();
        if (locals == NULL) {
            Py_DECREF(f);
            return NULL;
        }
        f->f_locals = locals;
    }
    else {
        if (locals == NULL)
            locals = globals;
        Py_INCREF(locals);
        f->f_locals = locals;
    }

    f->f_lasti = -1;
    f->f_lineno = code->co_firstlineno;
    f->f_iblock = 0;
    f->f_executing = 0;
    f->f_gen = NULL;
    f->f_trace_opcodes = 0;
    f->f_trace_lines = 1;

    return f;
}

```

从`extras = code->co_stacksize + code->co_nlocals + ncells + nfrees;` 中能看到栈帧的动态内存区由四个部分组成

而`ncells = PyTuple_GET_SIZE(code->co_cellvars);`，`nfrees = PyTuple_GET_SIZE(code->co_freevars);`我们知道额外的部分除了给`PyCodeObject`之外，还有给`co_cellvars`和`co_freevars`。

而其`f_valuestack`和`f_stacktop `分别表示栈底和栈顶

***

# PyFrameObject 的访问

我们可以很简单地通过 `sys.getframe()` 函数获取当前所运行的栈帧对象

***

# PyFrameObject 的 NAMESPACE

我们在`PyFrameObject`中能够看见的三个属性`*f_builtins`,`*f_globals`和`f_locals`分别代表了三个独立的命名空间NAMESPACE

在一般的python文件结构中，每个`.py`文件被视作一个module，我们把一个文件夹的所有`.py`文件看作一个模块，这个模块中有一个是主module，以`python main.py`进行加载，其余的用`import`引入。

而在`.py`文件里写的`import foo foo.bar`，`foo.bar`作为属性引用，使用了另外一个NAMESPACE中的名字

这种名字的查找是有规律的，Python遵照了[LEGB原则]()，即`locals -> enclosing function -> globals -> __builtins__`原则，以以下顺序依次查询

- locals 是函数内的名字空间，包括局部变量和形参
- enclosing 外部嵌套函数的名字空间（闭包中常见）
- globals 全局变量，函数定义所在模块的名字空间
- builtins 内置模块的名字空间

***

# 虚拟机运行框架

我们假设初始化已经完成，而现在第一个就是`PyEval_EvalFrameEx()`

```c
[Python/ceval.c]

PyObject *
PyEval_EvalFrameEx(PyFrameObject *f, int throwflag)
{
    PyThreadState *tstate = PyThreadState_GET();
    return tstate->interp->eval_frame(f, throwflag);
}
```

在这里面调用了`eval_frame()`,我们看它定义的宏

```c
/* Initialized to PyEval_EvalFrameDefault(). */
    _PyFrameEvalFunction eval_frame;
```

知道其是通过`PyEval_EvalFrameDefault()`函数调用的

但`PyEval_EvalFrameDefault()`的实现内容极多，一共几千行吧 :(

```c
[Python/ceval.c]

PyObject* _Py_HOT_FUNCTION _PyEval_EvalFrameDefault(PyFrameObject *f, int throwflag)
{
    ...
    co = f->f_code;
    names = co->co_names;
    consts = co->co_consts;
    fastlocals = f->f_localsplus;
    freevars = f->f_localsplus + co->co_nlocals;

    first_instr = (_Py_CODEUNIT *) PyBytes_AS_STRING(co->co_code);
    next_instr = first_instr;

    f->f_stacktop = NULL;       /* remains NULL unless yield suspends frame */
    f->f_executing = 1;
    ... 
}
```

虚拟机从头开始遍历整个`co_code`，`first_instr` 永远指向字节码指令序列的开始位置, `next_instr` 永远指向下一条待执行的字节码指令的位置, `f_lasti` 指向上一条已经执行过的字节码指令的位置

而整个架构就是`switch (opcode) {}`的`sitch/TARGET`结构

在函数最开始的时候还方便的设置了几个关于`tuple`获取和`code`移动的宏

```c
[Python/ceval.c/PyEval_EvalFrameDefault()]

/* Tuple access macros */

#ifndef Py_DEBUG
#define GETITEM(v, i) PyTuple_GET_ITEM((PyTupleObject *)(v), (i))
#else
#define GETITEM(v, i) PyTuple_GetItem((v), (i))
#endif

/* Code access macros */

/* The integer overflow is checked by an assertion below. */
#define INSTR_OFFSET()  \
    (sizeof(_Py_CODEUNIT) * (int)(next_instr - first_instr))
#define NEXTOPARG()  do { \
        _Py_CODEUNIT word = *next_instr; \
        opcode = _Py_OPCODE(word); \
        oparg = _Py_OPARG(word); \
        next_instr++; \
    } while (0)
#define JUMPTO(x)       (next_instr = first_instr + (x) / sizeof(_Py_CODEUNIT))
#define JUMPBY(x)       (next_instr += (x) / sizeof(_Py_CODEUNIT))
```

这里`NEXTOPARG()`中是用于读取下一条的，其读取操作码和参数

```c
[Include/code.h]
#ifdef WORDS_BIGENDIAN
#  define _Py_OPCODE(word) ((word) >> 8)
#  define _Py_OPARG(word) ((word) & 255)
#else
#  define _Py_OPCODE(word) ((word) & 255)
#  define _Py_OPARG(word) ((word) >> 8)
#endif
```

而关于其判断参数多少，在`switch(opcode)`之前有一个宏

```c
[Python/ceval.c/PyEval_EvalFrameDefault]

#ifdef LLTRACE
        /* Instruction tracing */

        if (lltrace) {
            if (HAS_ARG(opcode)) {
                printf("%d: %d, %d\n",
                       f->f_lasti, opcode, oparg);
            }
            else {
                printf("%d: %d\n",
                       f->f_lasti, opcode);
            }
        }
#endif

```

这里`HAS_ARG()`的宏定义在`[Include/opcode.h]`中，在我们上次讲到了，是一个判断操作数是否大于这个阈值的数值判断。

***

## 关于Why

在进入`switch(opcode)`选择后，每个选项中都有一个`why`变量，其实用于指示退出python引擎的状态因为有可能出错，下面是初始化函数`PyEval_EvalFrameDefault()`对于错误`why`的一些处理

```c
[Python/ceval.c/PyEval_EvalFrameDefault()]

error:

        assert(why == WHY_NOT);
        why = WHY_EXCEPTION;

        /* Double-check exception status. */
#ifdef NDEBUG
        if (!PyErr_Occurred())
            PyErr_SetString(PyExc_SystemError,
                            "error return without exception set");
#else
        assert(PyErr_Occurred());
#endif

        /* Log traceback info. */
        PyTraceBack_Here(f);

        if (tstate->c_tracefunc != NULL)
            call_exc_trace(tstate->c_tracefunc, tstate->c_traceobj,
                           tstate, f);

fast_block_end:
        assert(why != WHY_NOT);

        /* Unwind stacks if a (pseudo) exception occurred */
        while (why != WHY_NOT && f->f_iblock > 0) {
            /* Peek at the current block. */
            PyTryBlock *b = &f->f_blockstack[f->f_iblock - 1];

            assert(why != WHY_YIELD);
            if (b->b_type == SETUP_LOOP && why == WHY_CONTINUE) {
                why = WHY_NOT;
                JUMPTO(PyLong_AS_LONG(retval));
                Py_DECREF(retval);
                break;
            }
            /* Now we have to pop the block. */
            f->f_iblock--;

            if (b->b_type == EXCEPT_HANDLER) {
                UNWIND_EXCEPT_HANDLER(b);
                continue;
            }
            UNWIND_BLOCK(b);
            if (b->b_type == SETUP_LOOP && why == WHY_BREAK) {
                why = WHY_NOT;
                JUMPTO(b->b_handler);
                break;
            }
            if (why == WHY_EXCEPTION && (b->b_type == SETUP_EXCEPT
                || b->b_type == SETUP_FINALLY)) {
                PyObject *exc, *val, *tb;
                int handler = b->b_handler;
                _PyErr_StackItem *exc_info = tstate->exc_info;
                /* Beware, this invalidates all b->b_* fields */
                PyFrame_BlockSetup(f, EXCEPT_HANDLER, -1, STACK_LEVEL());
                PUSH(exc_info->exc_traceback);
                PUSH(exc_info->exc_value);
                if (exc_info->exc_type != NULL) {
                    PUSH(exc_info->exc_type);
                }
                else {
                    Py_INCREF(Py_None);
                    PUSH(Py_None);
                }
                PyErr_Fetch(&exc, &val, &tb);
                /* Make the raw exception data
                   available to the handler,
                   so a program can emulate the
                   Python main loop. */
                PyErr_NormalizeException(
                    &exc, &val, &tb);
                if (tb != NULL)
                    PyException_SetTraceback(val, tb);
                else
                    PyException_SetTraceback(val, Py_None);
                Py_INCREF(exc);
                exc_info->exc_type = exc;
                Py_INCREF(val);
                exc_info->exc_value = val;
                exc_info->exc_traceback = tb;
                if (tb == NULL)
                    tb = Py_None;
                Py_INCREF(tb);
                PUSH(tb);
                PUSH(val);
                PUSH(exc);
                why = WHY_NOT;
                JUMPTO(handler);
                break;
            }
            if (b->b_type == SETUP_FINALLY) {
                if (why & (WHY_RETURN | WHY_CONTINUE))
                    PUSH(retval);
                PUSH(PyLong_FromLong((long)why));
                why = WHY_NOT;
                JUMPTO(b->b_handler);
                break;
            }
        } /* unwind stack */

        /* End the loop if we still have an error (or return) */

        if (why != WHY_NOT)
            break;

        assert(!PyErr_Occurred());

    } /* main loop */

    assert(why != WHY_YIELD);
    /* Pop remaining stack entries. */
    while (!EMPTY()) {
        PyObject *o = POP();
        Py_XDECREF(o);
    }

    if (why != WHY_RETURN)
        retval = NULL;

    assert((retval != NULL) ^ (PyErr_Occurred() != NULL));

fast_yield:

    if (tstate->use_tracing) {
        if (tstate->c_tracefunc) {
            if (why == WHY_RETURN || why == WHY_YIELD) {
                if (call_trace(tstate->c_tracefunc, tstate->c_traceobj,
                               tstate, f,
                               PyTrace_RETURN, retval)) {
                    Py_CLEAR(retval);
                    why = WHY_EXCEPTION;
                }
            }
            else if (why == WHY_EXCEPTION) {
                call_trace_protected(tstate->c_tracefunc, tstate->c_traceobj,
                                     tstate, f,
                                     PyTrace_RETURN, NULL);
            }
        }
        if (tstate->c_profilefunc) {
            if (why == WHY_EXCEPTION)
                call_trace_protected(tstate->c_profilefunc,
                                     tstate->c_profileobj,
                                     tstate, f,
                                     PyTrace_RETURN, NULL);
            else if (call_trace(tstate->c_profilefunc, tstate->c_profileobj,
                                tstate, f,
                                PyTrace_RETURN, retval)) {
                Py_CLEAR(retval);
                /* why = WHY_EXCEPTION; useless yet but cause compiler warnings */
            }
        }
    }
```

我们可以看出来`why`会有不同的状态返回，而具体的状态码，也在`[Python/ceval.c]`中定义了

```c
[Python/ceval.c]

/* Status code for main loop (reason for stack unwind) */
enum why_code {
        WHY_NOT =       0x0001, /* No error */
        WHY_EXCEPTION = 0x0002, /* Exception occurred */
        WHY_RETURN =    0x0008, /* 'return' statement */
        WHY_BREAK =     0x0010, /* 'break' statement */
        WHY_CONTINUE =  0x0020, /* 'continue' statement */
        WHY_YIELD =     0x0040, /* 'yield' operator */
        WHY_SILENCED =  0x0080  /* Exception silenced by 'with' */
};
```

python框架总的流程就是，执行一个无尽的`for`循环，取出第一条字节码之后, 判断指令后执行, 然后一条接一条的从字节流中获取。

进行完异常检查后，最后进行出栈操作：

```c
[Python/ceval.c/PyEval_EvalFrameDefault()]

   /* pop frame */
exit_eval_frame:
    if (PyDTrace_FUNCTION_RETURN_ENABLED())
        dtrace_function_return(f);
    Py_LeaveRecursiveCall();
    f->f_executing = 0;
    tstate->frame = f->f_back;

    return _Py_CheckFunctionResult(NULL, retval, "PyEval_EvalFrameEx");
```

# Python的线程

可执行文件除开栈帧，还有很重要的：`进程`和`线程`

对于一个单进程多线程的可执行文件，操作系统会创建一个进程和多个线程，但线程键共享`global`变量，切换线程需要保存工作环境以便恢复。

python实现了对多线程的支持, 而且python中的线程就是操作系统上的一个原生线程。 python虚拟机是对CPU的抽象, 在切换线程之前同样需要保存关于当前线程的信息。

python是使用`PyThreadState`对象对线程信息进行抽象。

- 线程的抽象并非原始线程，python仍旧使用操作系统的原生线程
- 而对于线程抽象的操作，使用`PyInterpreterState`来操作
- python一般只维护一个`PyInterpreterState`，其中维护多个`PyThreadState`对象

而线程同步，少不了的就是锁机制「lock」，他们通过一个全局解释器锁 Global Interpreter Lock (GIL) 实现。

我们看`PyInterpreterState`和`PyThreadState`的实现：

```c
[Include/pystate.h]

typedef struct _is {

    struct _is *next;
    struct _ts *tstate_head;

    int64_t id;
    int64_t id_refcount;
    PyThread_type_lock id_mutex;

    PyObject *modules;
    PyObject *modules_by_index;
    PyObject *sysdict;
    PyObject *builtins;
    PyObject *importlib;

    /* Used in Python/sysmodule.c. */
    int check_interval;

    /* Used in Modules/_threadmodule.c. */
    long num_threads;
    /* Support for runtime thread stack size tuning.
       A value of 0 means using the platform's default stack size
       or the size specified by the THREAD_STACK_SIZE macro. */
    /* Used in Python/thread.c. */
    size_t pythread_stacksize;

    PyObject *codec_search_path;
    PyObject *codec_search_cache;
    PyObject *codec_error_registry;
    int codecs_initialized;
    int fscodec_initialized;

    _PyCoreConfig core_config;
    _PyMainInterpreterConfig config;
#ifdef HAVE_DLOPEN
    int dlopenflags;
#endif

    PyObject *builtins_copy;
    PyObject *import_func;
    /* Initialized to PyEval_EvalFrameDefault(). */
    _PyFrameEvalFunction eval_frame;

    Py_ssize_t co_extra_user_count;
    freefunc co_extra_freefuncs[MAX_CO_EXTRA_USERS];

#ifdef HAVE_FORK
    PyObject *before_forkers;
    PyObject *after_forkers_parent;
    PyObject *after_forkers_child;
#endif
    /* AtExit module */
    void (*pyexitfunc)(PyObject *);
    PyObject *pyexitmodule;

    uint64_t tstate_next_unique_id;
} PyInterpreterState;

```

```c
[Include/pystate.h]

typedef struct _ts {
    /* See Python/ceval.c for comments explaining most fields */

    struct _ts *prev;
    struct _ts *next;
    PyInterpreterState *interp;

    struct _frame *frame;
    int recursion_depth;
    char overflowed; /* The stack has overflowed. Allow 50 more calls
                        to handle the runtime error. */
    char recursion_critical; /* The current calls must not cause
                                a stack overflow. */
    int stackcheck_counter;

    /* 'tracing' keeps track of the execution depth when tracing/profiling.
       This is to prevent the actual trace/profile code from being recorded in
       the trace/profile. */
    int tracing;
    int use_tracing;

    Py_tracefunc c_profilefunc;
    Py_tracefunc c_tracefunc;
    PyObject *c_profileobj;
    PyObject *c_traceobj;

    /* The exception currently being raised */
    PyObject *curexc_type;
    PyObject *curexc_value;
    PyObject *curexc_traceback;

    /* The exception currently being handled, if no coroutines/generators
     * are present. Always last element on the stack referred to be exc_info.
     */
    _PyErr_StackItem exc_state;

    /* Pointer to the top of the stack of the exceptions currently
     * being handled */
    _PyErr_StackItem *exc_info;

    PyObject *dict;  /* Stores per-thread state */

    int gilstate_counter;

    PyObject *async_exc; /* Asynchronous exception to raise */
    unsigned long thread_id; /* Thread id where this tstate was created */

    int trash_delete_nesting;
    PyObject *trash_delete_later;

    /* Called when a thread state is deleted normally, but not when it
     * is destroyed after fork().
     * Pain:  to prevent rare but fatal shutdown errors (issue 18808),
     * Thread.join() must wait for the join'ed thread's tstate to be unlinked
     * from the tstate chain.  That happens at the end of a thread's life,
     * in pystate.c.
     * The obvious way doesn't quite work:  create a lock which the tstate
     * unlinking code releases, and have Thread.join() wait to acquire that
     * lock.  The problem is that we _are_ at the end of the thread's life:
     * if the thread holds the last reference to the lock, decref'ing the
     * lock will delete the lock, and that may trigger arbitrary Python code
     * if there's a weakref, with a callback, to the lock.  But by this time
     * _PyThreadState_Current is already NULL, so only the simplest of C code
     * can be allowed to run (in particular it must not be possible to
     * release the GIL).
     * So instead of holding the lock directly, the tstate holds a weakref to
     * the lock:  that's the value of on_delete_data below.  Decref'ing a
     * weakref is harmless.
     * on_delete points to _threadmodule.c's static release_sentinel() function.
     * After the tstate is unlinked, release_sentinel is called with the
     * weakref-to-lock (on_delete_data) argument, and release_sentinel releases
     * the indirectly held lock.
     */
    void (*on_delete)(void *);
    void *on_delete_data;

    int coroutine_origin_tracking_depth;

    PyObject *coroutine_wrapper;
    int in_coroutine_wrapper;

    PyObject *async_gen_firstiter;
    PyObject *async_gen_finalizer;

    PyObject *context;
    uint64_t context_ver;

    /* Unique thread state id. */
    uint64_t id;

    /* XXX signal handlers should also be here */

} PyThreadState;

```

明显的，每个进程下属一个解释器，下属一个线程池，进程间和线程间都有`next`属性通向下一个，而线程下维护一个栈帧表。