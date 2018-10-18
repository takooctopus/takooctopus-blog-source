---
title: 'python 内核分析（十一）：Python 虚拟机的表达式'
date: 2018-10-15 12:12:25
tags: PYTHON
categories: PYTHON # 分类
thumbnail: /assets/img/posts/PYTHON-KERNEL/default.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# 内建对象及其创建

我们定义一个简单的文件

```python
[foo.py]

i = 1
s = 'string'
l = []
d = {}

```

我们使用下面的语句访问

```bash
[pythonbash]

import dir

file = open('foo.py')
f = file.read()
c = compile(f, 'foo.py', 'exec')
d = dir(c)

for a in d:
    print(a,': ',getattr(c,a))

file.close()
```

最后我们得到最终的输出：

```bash
[pythonbash]

__class__ :  <class 'code'>
__delattr__ :  <method-wrapper '__delattr__' of code object at 0x6ffffe0d150>
__dir__ :  <built-in method __dir__ of code object at 0x6ffffe0d150>
__doc__ :  code(argcount, kwonlyargcount, nlocals, stacksize, flags, codestring,
      constants, names, varnames, filename, name, firstlineno,
      lnotab[, freevars[, cellvars]])

Create a code object.  Not for the faint of heart.
__eq__ :  <method-wrapper '__eq__' of code object at 0x6ffffe0d150>
__format__ :  <built-in method __format__ of code object at 0x6ffffe0d150>
__ge__ :  <method-wrapper '__ge__' of code object at 0x6ffffe0d150>
__getattribute__ :  <method-wrapper '__getattribute__' of code object at 0x6ffff                                                                              e0d150>
__gt__ :  <method-wrapper '__gt__' of code object at 0x6ffffe0d150>
__hash__ :  <method-wrapper '__hash__' of code object at 0x6ffffe0d150>
__init__ :  <method-wrapper '__init__' of code object at 0x6ffffe0d150>
__init_subclass__ :  <built-in method __init_subclass__ of type object at 0x5575                                                                              cbf40>
__le__ :  <method-wrapper '__le__' of code object at 0x6ffffe0d150>
__lt__ :  <method-wrapper '__lt__' of code object at 0x6ffffe0d150>
__ne__ :  <method-wrapper '__ne__' of code object at 0x6ffffe0d150>
__new__ :  <built-in method __new__ of type object at 0x5575cbf40>
__reduce__ :  <built-in method __reduce__ of code object at 0x6ffffe0d150>
__reduce_ex__ :  <built-in method __reduce_ex__ of code object at 0x6ffffe0d150>
__repr__ :  <method-wrapper '__repr__' of code object at 0x6ffffe0d150>
__setattr__ :  <method-wrapper '__setattr__' of code object at 0x6ffffe0d150>
__sizeof__ :  <built-in method __sizeof__ of code object at 0x6ffffe0d150>
__str__ :  <method-wrapper '__str__' of code object at 0x6ffffe0d150>
__subclasshook__ :  <built-in method __subclasshook__ of type object at 0x5575cb                                                                              f40>
co_argcount :  0
co_cellvars :  ()
co_code :  b'd\x00Z\x00d\x01Z\x01g\x00Z\x02i\x00Z\x03d\x02S\x00'
co_consts :  (1, 'string', None)
co_filename :  foo.py
co_firstlineno :  1
co_flags :  64
co_freevars :  ()
co_kwonlyargcount :  0
co_lnotab :  b'\x04\x01\x04\x01\x04\x01'
co_name :  <module>
co_names :  ('i', 's', 'l', 'd')
co_nlocals :  0
co_stacksize :  1
co_varnames :  ()

```

除开这些属性后，我们再去看这段代码的字节码:

```bash
[pythonbash]

import dis
dis.dis(c)

output: 

  1           0 LOAD_CONST               0 (1)
              2 STORE_NAME               0 (i)

  2           4 LOAD_CONST               1 ('string')
              6 STORE_NAME               1 (s)

  3           8 BUILD_LIST               0
             10 STORE_NAME               2 (l)

  4          12 BUILD_MAP                0
             14 STORE_NAME               3 (d)
             16 LOAD_CONST               2 (None)
             18 RETURN_VALUE

```

上面就是`foo.py`所生成的字节码

## 可能用到的宏

```c
[Python/ceval.c]

// 获取tuple中的元素
#define GETITEM(v, i) PyTuple_GET_ITEM((PyTupleObject *)(v), (i))

// 调整栈顶指针
#define BASIC_STACKADJ(n) (stack_pointer += n)

// 入栈操作
#define BASIC_PUSH(v)     (*stack_pointer++ = (v))
#define PUSH(v)                BASIC_PUSH(v)

// 出栈操作
#define BASIC_POP()       (*--stack_pointer)
#define POP()                  BASIC_POP()
```

***
## 指令解析

对应第一行

```bash
[pythonbash]

  1           0 LOAD_CONST               0 (1)
              2 STORE_NAME               0 (i)

```

同样的，我们去`[Python/ceval.c/_PyEval_EvalFrameDefault()]`中寻找:

```c
[Python/ceval.c/_PyEval_EvalFrameDefault()]

PREDICTED(LOAD_CONST);
TARGET(LOAD_CONST) {
    PyObject *value = GETITEM(consts, oparg);
    Py_INCREF(value);
    PUSH(value);
    FAST_DISPATCH();
}
```

其上半句是取出常数的，`GETITEM()`相当于`GETITEM(consts,1)`,并将这个对象压入时栈。

假设当前时栈对象`PyFrameObject`是`f`，`consts`就相当于`f->f_code->co_consts`。

在压入时栈后，虚拟机就需要为`local`命名空间中创造一个与其相映射的符号`i`。

```c
[Python/ceval.c/_PyEval_EvalFrameDefault()]

TARGET(STORE_NAME) {
            PyObject *name = GETITEM(names, oparg);
            PyObject *v = POP();
            PyObject *ns = f->f_locals;
            int err;
            if (ns == NULL) {
                PyErr_Format(PyExc_SystemError,
                             "no locals found when storing %R", name);
                Py_DECREF(v);
                goto error;
            }
            if (PyDict_CheckExact(ns))
                err = PyDict_SetItem(ns, name, v);
            else
                err = PyObject_SetItem(ns, name, v);
            Py_DECREF(v);
            if (err != 0)
                goto error;
            DISPATCH();
        }

```

我们可以看见`names`也就是`f->f_code->co_names`即符号表

`ns`是`f->f_locals`是本地命名空间，而`NAMESPACE`都是一个`dict`对象，保存了一个变量和值的映射关系。

***

第二行对于字符串变量的赋值也差不多，仅仅是索引的变化

```bash
[pythonbash]

  2           4 LOAD_CONST               1 ('string')
              6 STORE_NAME               1 (s)

```

***

第三行`l = []` 创建了一个list

```bash
[pythonbash]

  3           8 BUILD_LIST               0
             10 STORE_NAME               2 (l)

```

其中调用的是`BUILD_LIST`：

```c
[Python/ceval.c/_PyEval_EvalFrameDefault()]

TARGET(BUILD_LIST) {
            PyObject *list =  PyList_New(oparg);
            if (list == NULL)
                goto error;
            while (--oparg >= 0) {
                PyObject *item = POP();
                PyList_SET_ITEM(list, oparg, item);
            }
            PUSH(list);
            DISPATCH();
        }
```

这里会新建一个`list`，并在如果参数为空的时候直接压入栈中，反之如果参数不为空，就会从栈中弹出元素，加入这个list。

***

第四行的`d = {}`会创建一个`dict`，

```bash
[pythonbash]

  4          12 BUILD_MAP                0
             14 STORE_NAME               3 (d)
             16 LOAD_CONST               2 (None)
             18 RETURN_VALUE

```

我们看`BUILD_MAP`所对应的`case`


```c
[Python/ceval.c/_PyEval_EvalFrameDefault()]

TARGET(BUILD_MAP) {
            Py_ssize_t i;
            PyObject *map = _PyDict_NewPresized((Py_ssize_t)oparg);
            if (map == NULL)
                goto error;
            for (i = oparg; i > 0; i--) {
                int err;
                PyObject *key = PEEK(2*i);
                PyObject *value = PEEK(2*i - 1);
                err = PyDict_SetItem(map, key, value);
                if (err != 0) {
                    Py_DECREF(map);
                    goto error;
                }
            }

            while (oparg--) {
                Py_DECREF(POP());
                Py_DECREF(POP());
            }
            PUSH(map);
            DISPATCH();
        }

```

***

当执行完这一段`Code Block`后，python需要返回一些值

```c
[Python/ceval.c/_PyEval_EvalFrameDefault()]

TARGET(RETURN_VALUE) {
    retval = POP();
    why = WHY_RETURN;
    goto fast_block_end;
}

```
将其实际返回的值放在`retval`中，`POP()`从运行时栈中拉出的，实际值是上次`LOAD_CONST               2 (None)`的，即最后返回一个`None`值。最后跳转至`fast_block_end`中。

```c

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

```

将虚拟机状态设置为`WHY_RETURN`，进入最后end阶段

# 复杂内建对象的创建

我们考虑建立一个新的非空`list`或者`dict`

```python
[bar.py]

l = [1, 2]
d = {"3": 3, "4": 4}

```

我们同样的采用下面的代码查看：

```bash
[pythonbash]

import dir

file = open('bar.py')
f = file.read()
c = compile(f, 'bar.py', 'exec')
d = dir(c)

# 打印信息
for a in d:
    print(a,': ',getattr(c,a))

file.close()

# 显示字节码
import dis
dis.dis(c)

```

我们查看结果:

```bash
[pythonbash]

#对象信息
__class__ :  <class 'code'>
__delattr__ :  <method-wrapper '__delattr__' of code object at 0x6ffffeb1c90>
__dir__ :  <built-in method __dir__ of code object at 0x6ffffeb1c90>
__doc__ :  code(argcount, kwonlyargcount, nlocals, stacksize, flags, codestring,
      constants, names, varnames, filename, name, firstlineno,
      lnotab[, freevars[, cellvars]])

Create a code object.  Not for the faint of heart.
__eq__ :  <method-wrapper '__eq__' of code object at 0x6ffffeb1c90>
__format__ :  <built-in method __format__ of code object at 0x6ffffeb1c90>
__ge__ :  <method-wrapper '__ge__' of code object at 0x6ffffeb1c90>
__getattribute__ :  <method-wrapper '__getattribute__' of code object at 0x6ffff                                                                              eb1c90>
__gt__ :  <method-wrapper '__gt__' of code object at 0x6ffffeb1c90>
__hash__ :  <method-wrapper '__hash__' of code object at 0x6ffffeb1c90>
__init__ :  <method-wrapper '__init__' of code object at 0x6ffffeb1c90>
__init_subclass__ :  <built-in method __init_subclass__ of type object at 0x5575                                                                              cbf40>
__le__ :  <method-wrapper '__le__' of code object at 0x6ffffeb1c90>
__lt__ :  <method-wrapper '__lt__' of code object at 0x6ffffeb1c90>
__ne__ :  <method-wrapper '__ne__' of code object at 0x6ffffeb1c90>
__new__ :  <built-in method __new__ of type object at 0x5575cbf40>
__reduce__ :  <built-in method __reduce__ of code object at 0x6ffffeb1c90>
__reduce_ex__ :  <built-in method __reduce_ex__ of code object at 0x6ffffeb1c90>
__repr__ :  <method-wrapper '__repr__' of code object at 0x6ffffeb1c90>
__setattr__ :  <method-wrapper '__setattr__' of code object at 0x6ffffeb1c90>
__sizeof__ :  <built-in method __sizeof__ of code object at 0x6ffffeb1c90>
__str__ :  <method-wrapper '__str__' of code object at 0x6ffffeb1c90>
__subclasshook__ :  <built-in method __subclasshook__ of type object at 0x5575cb                                                                              f40>
co_argcount :  0
co_cellvars :  ()
co_code :  b'd\x00d\x01g\x02Z\x00d\x00d\x02d\x03\x9c\x02Z\x01d\x04S\x00'
co_consts :  (1, 2, 3, ('1', '3'), None)
co_filename :  bar.py
co_firstlineno :  1
co_flags :  64
co_freevars :  ()
co_kwonlyargcount :  0
co_lnotab :  b'\x08\x01'
co_name :  <module>
co_names :  ('l', 'd')
co_nlocals :  0
co_stacksize :  3
co_varnames :  ()

```
我们可以看见常量表`co_const`和符号表`co_names`，注意常量表中的小整数是共用的，大整数不会。

以及其字节码

```bash
[pythonbash]

output:

  1           0 LOAD_CONST               0 (1)
              2 LOAD_CONST               1 (2)
              4 BUILD_LIST               2
              6 STORE_NAME               0 (l)

  2           8 LOAD_CONST               0 (1)
             10 LOAD_CONST               2 (3)
             12 LOAD_CONST               3 (('1', '3'))
             14 BUILD_CONST_KEY_MAP      2
             16 STORE_NAME               1 (d)
             18 LOAD_CONST               4 (None)
             20 RETURN_VALUE

```

注意，在创建`list`时，即调用`BUILD_LIST`时传入参数`2`，之后会从栈帧中读出两个元素。

而创建`dict`时，会先将其值压入栈，再将其`key`压入栈，最后才创建`dict`。

我们看创建函数`BUILD_CONST_KEY_MAP`

```c
[Python/ceval.c/_PyEval_EvalFrameDefault()]

TARGET(BUILD_CONST_KEY_MAP) {
            Py_ssize_t i;
            PyObject *map;
            PyObject *keys = TOP();
            if (!PyTuple_CheckExact(keys) ||
                PyTuple_GET_SIZE(keys) != (Py_ssize_t)oparg) {
                PyErr_SetString(PyExc_SystemError,
                                "bad BUILD_CONST_KEY_MAP keys argument");
                goto error;
            }
            map = _PyDict_NewPresized((Py_ssize_t)oparg);
            if (map == NULL) {
                goto error;
            }
            for (i = oparg; i > 0; i--) {
                int err;
                PyObject *key = PyTuple_GET_ITEM(keys, oparg - i);
                PyObject *value = PEEK(i + 1);
                err = PyDict_SetItem(map, key, value);
                if (err != 0) {
                    Py_DECREF(map);
                    goto error;
                }
            }

            Py_DECREF(POP());
            while (oparg--) {
                Py_DECREF(POP());
            }
            PUSH(map);
            DISPATCH();
        }

```

***

# 其他的一般表达式

这里继续使用了一段代码来看其实现：

```python
[foo.py]
a = 5
b = a 
c = a + b
print(c)

```

```bash
[pythonbash]

__class__ :  <class 'code'>
__delattr__ :  <method-wrapper '__delattr__' of code object at 0x6ffffeb1c90>
__dir__ :  <built-in method __dir__ of code object at 0x6ffffeb1c90>
__doc__ :  code(argcount, kwonlyargcount, nlocals, stacksize, flags, codestring,
      constants, names, varnames, filename, name, firstlineno,
      lnotab[, freevars[, cellvars]])

Create a code object.  Not for the faint of heart.
__eq__ :  <method-wrapper '__eq__' of code object at 0x6ffffeb1c90>
__format__ :  <built-in method __format__ of code object at 0x6ffffeb1c90>
__ge__ :  <method-wrapper '__ge__' of code object at 0x6ffffeb1c90>
__getattribute__ :  <method-wrapper '__getattribute__' of code object at 0x6ffffeb1c90>
__gt__ :  <method-wrapper '__gt__' of code object at 0x6ffffeb1c90>
__hash__ :  <method-wrapper '__hash__' of code object at 0x6ffffeb1c90>
__init__ :  <method-wrapper '__init__' of code object at 0x6ffffeb1c90>
__init_subclass__ :  <built-in method __init_subclass__ of type object at 0x5575cbf40>
__le__ :  <method-wrapper '__le__' of code object at 0x6ffffeb1c90>
__lt__ :  <method-wrapper '__lt__' of code object at 0x6ffffeb1c90>
__ne__ :  <method-wrapper '__ne__' of code object at 0x6ffffeb1c90>
__new__ :  <built-in method __new__ of type object at 0x5575cbf40>
__reduce__ :  <built-in method __reduce__ of code object at 0x6ffffeb1c90>
__reduce_ex__ :  <built-in method __reduce_ex__ of code object at 0x6ffffeb1c90>
__repr__ :  <method-wrapper '__repr__' of code object at 0x6ffffeb1c90>
__setattr__ :  <method-wrapper '__setattr__' of code object at 0x6ffffeb1c90>
__sizeof__ :  <built-in method __sizeof__ of code object at 0x6ffffeb1c90>
__str__ :  <method-wrapper '__str__' of code object at 0x6ffffeb1c90>
__subclasshook__ :  <built-in method __subclasshook__ of type object at 0x5575cbf40>
co_argcount :  0
co_cellvars :  ()
co_code :  b'd\x00Z\x00e\x00Z\x01e\x00e\x01\x17\x00Z\x02e\x03e\x02\x83\x01\x01\x00d\x01S\x00'
co_consts :  (5, None)
co_filename :  foo.py
co_firstlineno :  1
co_flags :  64
co_freevars :  ()
co_kwonlyargcount :  0
co_lnotab :  b'\x04\x01\x04\x01\x08\x01'
co_name :  <module>
co_names :  ('a', 'b', 'c', 'print')
co_nlocals :  0
co_stacksize :  2
co_varnames :  ()

```

以及其字节码

```bash
[pythonbash]

  1           0 LOAD_CONST               0 (5)
              2 STORE_NAME               0 (a)

  2           4 LOAD_NAME                0 (a)
              6 STORE_NAME               1 (b)

  3           8 LOAD_NAME                0 (a)
             10 LOAD_NAME                1 (b)
             12 BINARY_ADD
             14 STORE_NAME               2 (c)

  4          16 LOAD_NAME                3 (print)
             18 LOAD_NAME                2 (c)
             20 CALL_FUNCTION            1
             22 POP_TOP
             24 LOAD_CONST               1 (None)
             26 RETURN_VALUE

```

## 符号搜索

首先是一个`LOAD_NAME`这个指令，其会从符号表中取出变量值并压入栈中。

```c
[Python/ceval.c/_PyEval_EvalFrameDefault()]

TARGET(LOAD_NAME) {
            PyObject *name = GETITEM(names, oparg);
            PyObject *locals = f->f_locals;
            PyObject *v;
            if (locals == NULL) {
                PyErr_Format(PyExc_SystemError,
                             "no locals when loading %R", name);
                goto error;
            }
            if (PyDict_CheckExact(locals)) {
                v = PyDict_GetItem(locals, name);
                Py_XINCREF(v);
            }
            else {
                v = PyObject_GetItem(locals, name);
                if (v == NULL) {
                    if (!PyErr_ExceptionMatches(PyExc_KeyError))
                        goto error;
                    PyErr_Clear();
                }
            }
            if (v == NULL) {
                v = PyDict_GetItem(f->f_globals, name);
                Py_XINCREF(v);
                if (v == NULL) {
                    if (PyDict_CheckExact(f->f_builtins)) {
                        v = PyDict_GetItem(f->f_builtins, name);
                        if (v == NULL) {
                            format_exc_check_arg(
                                        PyExc_NameError,
                                        NAME_ERROR_MSG, name);
                            goto error;
                        }
                        Py_INCREF(v);
                    }
                    else {
                        v = PyObject_GetItem(f->f_builtins, name);
                        if (v == NULL) {
                            if (PyErr_ExceptionMatches(PyExc_KeyError))
                                format_exc_check_arg(
                                            PyExc_NameError,
                                            NAME_ERROR_MSG, name);
                            goto error;
                        }
                    }
                }
            }
            PUSH(v);
            DISPATCH();
        }

```

我们看到其采用了`LGB`原则去命名空间搜索变量。

## 数值运算

数值相加是指令`BINARY_ADD`，其会先将`a`和`b`取出「`LOAD_NAME`」压入时栈，再进行加法

```c
[Python/ceval.c/_PyEval_EvalFrameDefault()]

TARGET(BINARY_ADD) {
            PyObject *right = POP();
            PyObject *left = TOP();
            PyObject *sum;
            /* NOTE(haypo): Please don't try to micro-optimize int+int on
               CPython using bytecode, it is simply worthless.
               See http://bugs.python.org/issue21955 and
               http://bugs.python.org/issue10044 for the discussion. In short,
               no patch shown any impact on a realistic benchmark, only a minor
               speedup on microbenchmarks. */
            if (PyUnicode_CheckExact(left) &&
                     PyUnicode_CheckExact(right)) {
                sum = unicode_concatenate(left, right, f, next_instr);
                /* unicode_concatenate consumed the ref to left */
            }
            else {
                sum = PyNumber_Add(left, right);
                Py_DECREF(left);
            }
            Py_DECREF(right);
            SET_TOP(sum);
            if (sum == NULL)
                goto error;
            DISPATCH();
        }

```

会判断左右类型，如果为字符串就进行字符串拼接，如果不是就进行`PyNumber_Add`进行数值相加。

## 信息输出

而`print()`会先将`print`和`c`压入栈，再调用`CALL_FUNCTION`，其参数为`1`

```c
[Python/ceval.c/_PyEval_EvalFrameDefault()]

PREDICTED(CALL_FUNCTION);
        TARGET(CALL_FUNCTION) {
            PyObject **sp, *res;
            sp = stack_pointer;
            res = call_function(&sp, oparg, NULL);
            stack_pointer = sp;
            PUSH(res);
            if (res == NULL) {
                goto error;
            }
            DISPATCH();
        }
```

我们可以看见其模拟了一个CPU的过程，使用`call_function`调用

其中会进行类型检查等一系列动作，最后调用`builtin_print()`函数输出