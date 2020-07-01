---
title: 'python 内核分析（八）：Pyc与PyCodeObject'
date: 2018-10-10 10:17:34
tags: 
- CS
- PYTHON
categories: CS # 分类
thumbnail: /assets/img/posts/CS/PythonKernel.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# 关于pyc文件

在python运行源程序时，首先要经过编译，其会变为字节码文件，即我们所说的`.pyc`文件。但其中主要的是`PyCodeObject`。

```c
[Include/code.h]

/* Bytecode object */
typedef struct {
    PyObject_HEAD
    int co_argcount;            /* #arguments, except *args 此处的参数个数*/
    int co_kwonlyargcount;      /* #keyword only arguments 关键参数*/
    int co_nlocals;             /* #local variables 作用域中的局部变量*/
    int co_stacksize;           /* #entries needed for evaluation stack 这段code所需要的栈空间*/
    int co_flags;               /* CO_..., see below */
    int co_firstlineno;         /* first source line number 对应.py文件的起始行*/
    PyObject *co_code;          /* instruction opcodes 编译的字节码，为PyByteObject*/
    PyObject *co_consts;        /* list (constants used) 所有常量，PyTupleObject*/
    PyObject *co_names;         /* list of strings (names used) 所有符号，PyTupleObject*/
    PyObject *co_varnames;      /* tuple of strings (local variable names) 局部变量名 */
    PyObject *co_freevars;      /* tuple of strings (free variable names) 闭包函数相关变量*/
    PyObject *co_cellvars;      /* tuple of strings (cell variable names) 嵌套函数相关局部变量*/
    /* The rest aren't used in either hash or comparisons, except for co_name,
       used in both. This is done to preserve the name and line number
       for tracebacks and debuggers; otherwise, constant de-duplication
       would collapse identical functions/lambdas defined on different lines.
    */
    Py_ssize_t *co_cell2arg;    /* Maps cell vars which are arguments. */
    PyObject *co_filename;      /* unicode (where it was loaded from) 源文件名*/
    PyObject *co_name;          /* unicode (name, for reference) 引用所使用的名称*/
    PyObject *co_lnotab;        /* string (encoding addr<->lineno mapping) See
                                   Objects/lnotab_notes.txt for details. 字节码指令与.py行号所对应的映射关系*/
    void *co_zombieframe;       /* for optimization only (see frameobject.c) */
    PyObject *co_weakreflist;   /* to support weakrefs to code objects */
    /* Scratch space for extra data relating to the code object.
       Type is a void* to keep the format private in codeobject.c to force
       people to go through the proper APIs. */
    void *co_extra;
} PyCodeObject;
```

编译器对源码进行编译时，对于每个代码域，都会创建一个`PyCodeObject`与之对应，即文件、类或者函数。

在你运行python，使用编译语句的时候，返回的对象能直接通过这些属性访问

```python
   codeObject = compile('foo.py','foo.py','exec')
   type(co)
   co.co_names
```

这里面我们就能直接得到code的属性

你要将Object写入文件，就需要

```c
[Python/marshal.c]

void
PyMarshal_WriteObjectToFile(PyObject *x, FILE *fp, int version)
{
    char buf[BUFSIZ];
    WFILE wf;
    memset(&wf, 0, sizeof(wf));
    wf.fp = fp;
    wf.ptr = wf.buf = buf;
    wf.end = wf.ptr + sizeof(buf);
    wf.error = WFERR_OK;
    wf.version = version;
    if (w_init_refs(&wf, version))
        return; /* caller mush check PyErr_Occurred() */
    w_object(x, &wf);
    w_clear_refs(&wf);
    w_flush(&wf);
}
```

其中`WFILE`结构体定义

```c
[Python/marshal.c]

typedef struct {
    FILE *fp;
    int error;  /* see WFERR_* values */
    int depth;
    PyObject *str;
    char *ptr;
    char *end;
    char *buf;
    _Py_hashtable_t *hashtable;
    int version;
} WFILE;
```

其中`fp`指向了最后要写入的文件。

在确定了写入的文件后，就需要向文件中写入对象：

```c
[Python/marshal.c]

static void
w_object(PyObject *v, WFILE *p)
{
    char flag = '\0';

    p->depth++;

    if (p->depth > MAX_MARSHAL_STACK_DEPTH) {
        p->error = WFERR_NESTEDTOODEEP;
    }
    else if (v == NULL) {
        w_byte(TYPE_NULL, p);
    }
    else if (v == Py_None) {
        w_byte(TYPE_NONE, p);
    }
    else if (v == PyExc_StopIteration) {
        w_byte(TYPE_STOPITER, p);
    }
    else if (v == Py_Ellipsis) {
        w_byte(TYPE_ELLIPSIS, p);
    }
    else if (v == Py_False) {
        w_byte(TYPE_FALSE, p);
    }
    else if (v == Py_True) {
        w_byte(TYPE_TRUE, p);
    }
    else if (!w_ref(v, &flag, p))
        w_complex_object(v, flag, p);

    p->depth--;
}
```

一般的，调用了`w_byte()`函数进行写入缓冲区，`w_flush()`将缓冲区`p->buf`写入文件

```c
[Python/marshal.c]

#define w_byte(c, p) do {                               \
        if ((p)->ptr != (p)->end || w_reserve((p), 1))  \
            *(p)->ptr++ = (c);                          \
    } while(0)

static void
w_flush(WFILE *p)
{
    assert(p->fp != NULL);
    fwrite(p->buf, 1, p->ptr - p->buf, p->fp);
    p->ptr = p->buf;
}

static int
w_reserve(WFILE *p, Py_ssize_t needed)
{
    Py_ssize_t pos, size, delta;
    if (p->ptr == NULL)
        return 0; /* An error already occurred */
    if (p->fp != NULL) {
        w_flush(p);
        return needed <= p->end - p->ptr;
    }
    assert(p->str != NULL);
    pos = p->ptr - p->buf;
    size = PyBytes_Size(p->str);
    if (size > 16*1024*1024)
        delta = (size >> 3);            /* 12.5% overallocation */
    else
        delta = size + 1024;
    delta = Py_MAX(delta, needed);
    if (delta > PY_SSIZE_T_MAX - size) {
        p->error = WFERR_NOMEMORY;
        return 0;
    }
    size += delta;
    if (_PyBytes_Resize(&p->str, size) != 0) {
        p->ptr = p->buf = p->end = NULL;
        return 0;
    }
    else {
        p->buf = PyBytes_AS_STRING(p->str);
        p->ptr = p->buf + pos;
        p->end = p->buf + size;
        return 1;
    }
}
```

特别的，我们在调用`w_byte()`时，需要传入数据类型。

```c

#define TYPE_NULL               '0'
#define TYPE_NONE               'N'
#define TYPE_FALSE              'F'
#define TYPE_TRUE               'T'
#define TYPE_STOPITER           'S'
#define TYPE_ELLIPSIS           '.'
#define TYPE_INT                'i'
/* TYPE_INT64 is not generated anymore.
   Supported for backward compatibility only. */
#define TYPE_INT64              'I'
#define TYPE_FLOAT              'f'
#define TYPE_BINARY_FLOAT       'g'
#define TYPE_COMPLEX            'x'
#define TYPE_BINARY_COMPLEX     'y'
#define TYPE_LONG               'l'
#define TYPE_STRING             's'
#define TYPE_INTERNED           't'
#define TYPE_REF                'r'
#define TYPE_TUPLE              '('
#define TYPE_LIST               '['
#define TYPE_DICT               '{'
#define TYPE_CODE               'c'
#define TYPE_UNICODE            'u'
#define TYPE_UNKNOWN            '?'
#define TYPE_SET                '<'
#define TYPE_FROZENSET          '>'
#define FLAG_REF                '\x80' /* with a type, add obj to index */

#define TYPE_ASCII              'a'
#define TYPE_ASCII_INTERNED     'A'
#define TYPE_SMALL_TUPLE        ')'
#define TYPE_SHORT_ASCII        'z'
#define TYPE_SHORT_ASCII_INTERNED 'Z'
```

因为写入文件时，所有数据都变为了字节流，我们通过开始时的这个标志，能得知一个新的对象的开始。

# PyCodeObject的加载

其加载过程开始在`PyMarshal_ReadObjectFromFile()`函数：

```c
[Python/marshal.c]

PyObject *
PyMarshal_ReadObjectFromFile(FILE *fp)
{
    RFILE rf;
    PyObject *result;
    rf.fp = fp;
    rf.readable = NULL;
    rf.depth = 0;
    rf.ptr = rf.end = NULL;
    rf.buf = NULL;
    rf.refs = PyList_New(0);
    if (rf.refs == NULL)
        return NULL;
    result = r_object(&rf);
    Py_DECREF(rf.refs);
    if (rf.buf != NULL)
        PyMem_FREE(rf.buf);
    return result;
}
```

实际的读入在`r_object()`函数中：

```c
[Python/marshal.c]

static PyObject *
r_object(RFILE *p)
{
    /* NULL is a valid return value, it does not necessarily means that
       an exception is set. */
    PyObject *v, *v2;
    Py_ssize_t idx = 0;
    long i, n;
    int type, code = r_byte(p);
    int flag, is_interned = 0;
    PyObject *retval = NULL;

    if (code == EOF) {
        PyErr_SetString(PyExc_EOFError,
                        "EOF read where object expected");
        return NULL;
    }

    p->depth++;

    if (p->depth > MAX_MARSHAL_STACK_DEPTH) {
        p->depth--;
        PyErr_SetString(PyExc_ValueError, "recursion limit exceeded");
        return NULL;
    }

    flag = code & FLAG_REF;
    type = code & ~FLAG_REF;

#define R_REF(O) do{\
    if (flag) \
        O = r_ref(O, flag, p);\
} while (0)

    switch (type) {

    case TYPE_NULL:
        break;

    case TYPE_NONE:
        Py_INCREF(Py_None);
        retval = Py_None;
        break;

    case TYPE_STOPITER:
        Py_INCREF(PyExc_StopIteration);
        retval = PyExc_StopIteration;
        break;

    case TYPE_ELLIPSIS:
        Py_INCREF(Py_Ellipsis);
        retval = Py_Ellipsis;
        break;

    case TYPE_FALSE:
        Py_INCREF(Py_False);
        retval = Py_False;
        break;

    case TYPE_TRUE:
        Py_INCREF(Py_True);
        retval = Py_True;
        break;

    case TYPE_INT:
        n = r_long(p);
        retval = PyErr_Occurred() ? NULL : PyLong_FromLong(n);
        R_REF(retval);
        break;

    case TYPE_INT64:
        retval = r_long64(p);
        R_REF(retval);
        break;

    case TYPE_LONG:
        retval = r_PyLong(p);
        R_REF(retval);
        break;

    case TYPE_FLOAT:
        {
            char buf[256];
            const char *ptr;
            double dx;
            n = r_byte(p);
            if (n == EOF) {
                PyErr_SetString(PyExc_EOFError,
                    "EOF read where object expected");
                break;
            }
            ptr = r_string(n, p);
            if (ptr == NULL)
                break;
            memcpy(buf, ptr, n);
            buf[n] = '\0';
            dx = PyOS_string_to_double(buf, NULL, NULL);
            if (dx == -1.0 && PyErr_Occurred())
                break;
            retval = PyFloat_FromDouble(dx);
            R_REF(retval);
            break;
        }

        ...
        ...
        ...
        ...
        ...
        ...
        case TYPE_CODE:
        {
            int argcount;
            int kwonlyargcount;
            int nlocals;
            int stacksize;
            int flags;
            PyObject *code = NULL;
            PyObject *consts = NULL;
            PyObject *names = NULL;
            PyObject *varnames = NULL;
            PyObject *freevars = NULL;
            PyObject *cellvars = NULL;
            PyObject *filename = NULL;
            PyObject *name = NULL;
            int firstlineno;
            PyObject *lnotab = NULL;

            idx = r_ref_reserve(flag, p);
            if (idx < 0)
                break;

            v = NULL;

            /* XXX ignore long->int overflows for now */
            argcount = (int)r_long(p);
            if (PyErr_Occurred())
                goto code_error;
            kwonlyargcount = (int)r_long(p);
            if (PyErr_Occurred())
                goto code_error;
            nlocals = (int)r_long(p);
            if (PyErr_Occurred())
                goto code_error;
            stacksize = (int)r_long(p);
            if (PyErr_Occurred())
                goto code_error;
            flags = (int)r_long(p);
            if (PyErr_Occurred())
                goto code_error;
            code = r_object(p);
            if (code == NULL)
                goto code_error;
            consts = r_object(p);
            if (consts == NULL)
                goto code_error;
            names = r_object(p);
            if (names == NULL)
                goto code_error;
            varnames = r_object(p);
            if (varnames == NULL)
                goto code_error;
            freevars = r_object(p);
            if (freevars == NULL)
                goto code_error;
            cellvars = r_object(p);
            if (cellvars == NULL)
                goto code_error;
            filename = r_object(p);
            if (filename == NULL)
                goto code_error;
            name = r_object(p);
            if (name == NULL)
                goto code_error;
            firstlineno = (int)r_long(p);
            if (firstlineno == -1 && PyErr_Occurred())
                break;
            lnotab = r_object(p);
            if (lnotab == NULL)
                goto code_error;

            v = (PyObject *) PyCode_New(
                            argcount, kwonlyargcount,
                            nlocals, stacksize, flags,
                            code, consts, names, varnames,
                            freevars, cellvars, filename, name,
                            firstlineno, lnotab);
            v = r_ref_insert(v, idx, flag, p);

          code_error:
            Py_XDECREF(code);
            Py_XDECREF(consts);
            Py_XDECREF(names);
            Py_XDECREF(varnames);
            Py_XDECREF(freevars);
            Py_XDECREF(cellvars);
            Py_XDECREF(filename);
            Py_XDECREF(name);
            Py_XDECREF(lnotab);
        }
        retval = v;
        break;

    case TYPE_REF:
        n = r_long(p);
        if (n < 0 || n >= PyList_GET_SIZE(p->refs)) {
            if (n == -1 && PyErr_Occurred())
                break;
            PyErr_SetString(PyExc_ValueError, "bad marshal data (invalid reference)");
            break;
        }
        v = PyList_GET_ITEM(p->refs, n);
        if (v == Py_None) {
            PyErr_SetString(PyExc_ValueError, "bad marshal data (invalid reference)");
            break;
        }
        Py_INCREF(v);
        retval = v;
        break;

    default:
        /* Bogus data got written, which isn't ideal.
           This will let you keep working and recover. */
        PyErr_SetString(PyExc_ValueError, "bad marshal data (unknown type code)");
        break;

    }
    p->depth--;
    return retval;

```
