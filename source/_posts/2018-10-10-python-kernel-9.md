---
title: 'python 内核分析（九）：Pyc与字节码'
date: 2018-10-10 12:55:03
tags: PYTHON
categories: PYTHON # 分类
thumbnail: /assets/img/posts/PYTHON-KERNEL/1.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# Python字节码

我们知道Python虚拟机的实现机制是基于栈结构的，Cpython会使用三种类型的栈：

- 调用栈`call stack`。这是运行 `Python` 程序的主要结构。它为每个当前活动的函数调用使用了一个东西 —— “帧`frame`”，栈底是程序的入口点。每个函数调用推送一个新的帧到调用栈，每当函数调用返回后，这个帧被销毁。

- 在每个帧中，有一个计算栈`evaluation stack` （也称为数据栈`data stack`）。这个栈就是 Python 函数运行的地方，运行的 Python 代码大多数是由推入到这个栈中的东西组成的，操作它们，然后在返回后销毁它们。

- 在每个帧中，还有一个块栈`block stack`。它被 Python 用于去跟踪某些类型的控制结构：循环、`try / except` 块、以及 `with` 块，全部推入到块栈中，当你退出这些控制结构时，块栈被销毁。这将帮助 Python 了解任意给定时刻哪个块是活动的，比如，一个 `continu`e 或者 `break` 语句可能影响正确的块。


我们在`[Include/opcode.h]`中能找到关于这些字节码的定义：

```c
[Include/opcode.h]

 /* Instruction opcodes for compiled code */
#define POP_TOP                   1
#define ROT_TWO                   2
#define ROT_THREE                 3
#define DUP_TOP                   4
#define DUP_TOP_TWO               5
#define NOP                       9
#define UNARY_POSITIVE           10
#define UNARY_NEGATIVE           11
#define UNARY_NOT                12
#define UNARY_INVERT             15
#define BINARY_MATRIX_MULTIPLY   16
#define INPLACE_MATRIX_MULTIPLY  17
#define BINARY_POWER             19
#define BINARY_MULTIPLY          20
#define BINARY_MODULO            22
#define BINARY_ADD               23
#define BINARY_SUBTRACT          24
#define BINARY_SUBSCR            25
#define BINARY_FLOOR_DIVIDE      26
#define BINARY_TRUE_DIVIDE       27
#define INPLACE_FLOOR_DIVIDE     28
#define INPLACE_TRUE_DIVIDE      29
#define GET_AITER                50
#define GET_ANEXT                51
#define BEFORE_ASYNC_WITH        52
#define INPLACE_ADD              55
#define INPLACE_SUBTRACT         56
#define INPLACE_MULTIPLY         57
#define INPLACE_MODULO           59
#define STORE_SUBSCR             60
#define DELETE_SUBSCR            61
#define BINARY_LSHIFT            62
#define BINARY_RSHIFT            63
#define BINARY_AND               64
#define BINARY_XOR               65
#define BINARY_OR                66
#define INPLACE_POWER            67
#define GET_ITER                 68
#define GET_YIELD_FROM_ITER      69
#define PRINT_EXPR               70
#define LOAD_BUILD_CLASS         71
#define YIELD_FROM               72
#define GET_AWAITABLE            73
#define INPLACE_LSHIFT           75
#define INPLACE_RSHIFT           76
#define INPLACE_AND              77
#define INPLACE_XOR              78
#define INPLACE_OR               79
#define BREAK_LOOP               80
#define WITH_CLEANUP_START       81
#define WITH_CLEANUP_FINISH      82
#define RETURN_VALUE             83
#define IMPORT_STAR              84
#define SETUP_ANNOTATIONS        85
#define YIELD_VALUE              86
#define POP_BLOCK                87
#define END_FINALLY              88
#define POP_EXCEPT               89
#define HAVE_ARGUMENT            90
#define STORE_NAME               90
#define DELETE_NAME              91
#define UNPACK_SEQUENCE          92
#define FOR_ITER                 93
#define UNPACK_EX                94
#define STORE_ATTR               95
#define DELETE_ATTR              96
#define STORE_GLOBAL             97
#define DELETE_GLOBAL            98
#define LOAD_CONST              100
#define LOAD_NAME               101
#define BUILD_TUPLE             102
#define BUILD_LIST              103
#define BUILD_SET               104
#define BUILD_MAP               105
#define LOAD_ATTR               106
#define COMPARE_OP              107
#define IMPORT_NAME             108
#define IMPORT_FROM             109
#define JUMP_FORWARD            110
#define JUMP_IF_FALSE_OR_POP    111
#define JUMP_IF_TRUE_OR_POP     112
#define JUMP_ABSOLUTE           113
#define POP_JUMP_IF_FALSE       114
#define POP_JUMP_IF_TRUE        115
#define LOAD_GLOBAL             116
#define CONTINUE_LOOP           119
#define SETUP_LOOP              120
#define SETUP_EXCEPT            121
#define SETUP_FINALLY           122
#define LOAD_FAST               124
#define STORE_FAST              125
#define DELETE_FAST             126
#define RAISE_VARARGS           130
#define CALL_FUNCTION           131
#define MAKE_FUNCTION           132
#define BUILD_SLICE             133
#define LOAD_CLOSURE            135
#define LOAD_DEREF              136
#define STORE_DEREF             137
#define DELETE_DEREF            138
#define CALL_FUNCTION_KW        141
#define CALL_FUNCTION_EX        142
#define SETUP_WITH              143
#define EXTENDED_ARG            144
#define LIST_APPEND             145
#define SET_ADD                 146
#define MAP_ADD                 147
#define LOAD_CLASSDEREF         148
#define BUILD_LIST_UNPACK       149
#define BUILD_MAP_UNPACK        150
#define BUILD_MAP_UNPACK_WITH_CALL 151
#define BUILD_TUPLE_UNPACK      152
#define BUILD_SET_UNPACK        153
#define SETUP_ASYNC_WITH        154
#define FORMAT_VALUE            155
#define BUILD_CONST_KEY_MAP     156
#define BUILD_STRING            157
#define BUILD_TUPLE_UNPACK_WITH_CALL 158
#define LOAD_METHOD             160
#define CALL_METHOD             161

/* EXCEPT_HANDLER is a special, implicit block type which is created when
   entering an except handler. It is not an opcode but we define it here
   as we want it to be available to both frameobject.c and ceval.c, while
   remaining private.*/
#define EXCEPT_HANDLER 257
```

上面就是定义的几乎全部操作字节码了。

因为有些指令需要参数，所以定义了`HAVE_ARGUMENT`这个作为分界，没有参数的指令在它上面，编码小于它「90」,有参数的指令编码大于「90」

```c
[Include/opcode.h]
#define HAS_ARG(op) ((op) >= HAVE_ARGUMENT)
```

而任何代码域对象在函数中可以以属性`__code__` 来访问其`PyCodeObject`中的属性。

# Pyc文件的反编译

系统自带的函数 `dis.dis()` 将反汇编一个函数、方法、类、模块、编译过的 Python 代码对象、或者字符串包含的源代码，以及显示出一个人类可读的版本。

举例来说

```c
[foo.py]

x = 1
y = 2

def bar(x1, y1):
    return x1 + y1

z = bar(x, y)
print(z)
```

```bash

$ python -m dis foo.py
  1           0 LOAD_CONST               0 (1)
              2 STORE_NAME               0 (x)

  2           4 LOAD_CONST               1 (2)
              6 STORE_NAME               1 (y)

  4           8 LOAD_CONST               2 (<code object bar at 0x6ffffd5f390, file "foo.py", line 4>)
             10 LOAD_CONST               3 ('bar')
             12 MAKE_FUNCTION            0
             14 STORE_NAME               2 (bar)

  7          16 LOAD_NAME                2 (bar)
             18 LOAD_NAME                0 (x)
             20 LOAD_NAME                1 (y)
             22 CALL_FUNCTION            2
             24 STORE_NAME               3 (z)

  8          26 LOAD_NAME                4 (print)
             28 LOAD_NAME                3 (z)
             30 CALL_FUNCTION            1
             32 POP_TOP
             34 LOAD_CONST               4 (None)
             36 RETURN_VALUE

```

话说Python3.6之后，每条字节码不管有没有参数都占两字节，其中字节码指令和参数各占一字节了。
