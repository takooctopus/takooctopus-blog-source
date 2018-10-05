---
title: 博文翻译（一）：integer与pointer之间的区别
date: 2018-09-25 22:01:13
tags: Program
categories: Program # 分类
thumbnail: /assets/img/posts/Program-Tips/1.jpg
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对国外博客做的个人翻译，并在此附上原文地址
>[What's the difference between an integer and a pointer?](https://blog.regehr.org/archives/1621 "EMBEDDED IN ACADEMIA")

# integer与pointer之间的区别
(本文是原作者的一片[即将发表的论文](http://www.cs.utah.edu/~regehr/oopsla18.pdf "Reconciling High-Level Optimizations and Low-Level Code
in LLVM")的介绍和推广)

在一门汇编语言中我们通常情况下并不是很需要去担心integers与pointers之间的区别，有些指令恰好在其他指令进行运算的时候生成地址，虽然这里还有一个单数据类型：位向量。相反的，一种高级语言永远不会有机会弄混integer与pointer，因为在其实现中，抽象层上便是彼此隔绝的，并且显然高级语言可以选择不暴露任何与pointer类似的东西。

即我们讨论的有问题的情况，就是面向性能的系统编程语言: **C**, **C++** , **RUST** 以及一小部分其他的语言，他们的问题是：

- 1. 当可能的时候，他们会假装是高级的，用以去优化本来在汇编层面上可能不合理的问题。举个例子，他们假定了从一个堆中分配派生的指针不能为从不同堆分配派生的指针进行别名操作。
- 2. 而与此同时，当必要的时候，他们又允许编程人员在底层上操作指针，比如检查指针的表示，在一个已经存在的指针的附近（漂移）创建一个新的指针，在一个指针的未使用部分储存不相关的数据块，甚至在整个数据域外面创建指针。

这些目标之间的冲突：当一种语言在高级层面优化做得越好，它在系统核心（OS Kernal）上的表现就越不尽人意，反之亦然。

>我们有一些方法可以去解决这些：
    1. 我们可以通过高级语言去调用不同低级语言缩写的代码，比如Java在JNI中所做得那样
    2. 我们可以为一种安全语言创建一种特别的低级模式让它去打破普通模式，比如Rust在其非安全代码(unsafe code)中做得一样
    3. 我们可以不管它们，并且自由的混合高级与低级语言特性的代码，期望编译器能够将其分开，就像C和C++一样

## 我们举一个例子：

```c preset=tako-codeblock++
#include <stdio.h>
#include <stdlib.h>
 
int main(void) {
  int *x = malloc(4);
  *x = 3;
  int *y = malloc(4);
  *y = 5;
  uintptr_t xi = (uintptr_t)x;
  uintptr_t yi = (uintptr_t)y;
  uintptr_t diff = xi - yi;
  printf("diff = %ld\n", (long)diff);
  int *p = (int *)(yi + diff);
  *p = 7;
  printf("x = %p, *x = %d\n", x, *x);
  printf("y = %p, *y = %d\n", y, *y);
  printf("p = %p, *p = %d\n", p, *p);
}
```

当其运行在作者的电脑(MAC)上，其有运行结果

```bash preset=tako-commandline
$ clang-6.0 -O3 mem1d.c ; ./a.out
diff = -96
x = 0x7fcb0ec00300, *x = 7
y = 0x7fcb0ec00360, *y = 5
p = 0x7fcb0ec00300, *p = 7
$ gcc-8 -O3 mem1d.c ; ./a.out
diff = -96
x = 0x7f93b6400300, *x = 7
y = 0x7f93b6400360, *y = 5
p = 0x7f93b6400300, *p = 7
```

在这里发生了什么，第一，我们拿走了一些堆单元并初始化了他们的值。随后，我们使用了整型数学去创建了一个和x有着相同值得指针p，并且通过这个指针去做了存储操作。第三，我们向编译器去询问x,y以及p所指向的地址里面的值。GCC与Clang/LLVM均认为通过指针p做出的存储操作会改变x所指向的值，这个结果是我们所期望的，但让我们更深入地看一看。

首先，这个C程序是否执行了没有定义过的行为？并没有。计算越界的指针是未定义行为UB(Undefined Behaviour)但我们可以任意的对从那些指针中派生出来的整型值做我们想做的事。第二，C的标准中是否要求了我们在这里看到的行为？也没有。事实上它仅仅给出了关于从指针中派生出的整型数据的行为的指导。[你可以在这里看到来源](https://nullprogram.com/blog/2016/05/30/ "You Can't Always Hash Pointers in C")。

## 现在我们将上面的程序做一点小小的改动

```c preset=tako-codeblock
    #include <stdio.h>
    #include <stdlib.h>

    int main(void){
        int *x = malloc(4);
        *x = 3;
        int *y = malloc(4);
        *y = 5;
        uintptr_t xi = (uintptr_t)x;
        uintptr_t yi = (uintptr_t)y;
        uintptr_t diff = xi -yi;
        printf("diff = %ld\n", (long)diff);
        int *p = (int *)(yi - 96); // <--- CHANGED!
        *p = 7;
        printf("x = %p, *x = %d\n", x, *x);
        printf("y = %p, *y = %d\n", y, *y);
        printf("p = %p, *p = %d\n", p, *p);
    }
```

唯一的区别就在第13行里：取代我们之前加法中`diff`与`yi`相加的操作，我们显然使用一个显式值`-96`。现在我们运行程序：

```bash preset=tako-commandline
    $ clang-6.0 -O3 mem1d2.c ; ./a.out
    diff = -96
    x = 0x7ff21ac00300, *x = 7
    y = 0x7ff21ac00360, *y = 5
    p = 0x7ff21ac00300, *p = 7
    $ gcc-8 -O3 mem1d2.c ; ./a.out
    diff = -96
    x = 0x7fce5e400300, *x = 3
    y = 0x7fce5e400360, *y = 5
    p = 0x7fce5e400300, *p = 7
```

Clang 编译结果与之前的无异，但GCC不再认为通过p的赋值操作会导致x的值被复写了——那会导致x和p最终会有相同的值。这之间发生了什么，就GCC而言，因为我们计算x的地址时是通过估测而不是实际的观测。我们的估测实际上是非法的。（如果这个估测游戏对你来说太过于简单了，那么一些有趣的变化可以让你避免需要来自前一次程序运行的信息，比如估测地址时使用详尽的循环或进行大量的分配，来让生成的地址受到[鸽笼原理](https://en.wikipedia.org/wiki/Pigeonhole_principle "Pigeonhole principle")的限制）这儿我们再一次的在C标准之外运行良好并且我们更期望编译器作者能够做出决策以平衡优化能力以及开发者的期望两者间的矛盾。我喜欢这个例子，因为它有些奇怪但令人惊喜。

我们在这之中应该学到什么：第一，试图将一个编程语言指针认为其拥有和指针型整型值(pointer-sized integer value)拥有相同的规则，是错误的。即便是那些已经与未定义行为UB(Undefined Behaviour)达成一致的人也常常会惊讶于程序运行出的结果。第二，这些问题并不仅仅针对于 **C**和 **C++** ，而是会在任何想要高性能编译器且允许低级指针操作的编程语言中创造问题。

那么在Clang与GCC在实际编译时遵循的规律是什么呢？简单的回答就是它比较复杂且没有多少相关研究。一些有用的讨论可以在[这篇关于指针出处的文章](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2263.htm "Clarifying Pointer Provenance v4")中找到。如果想要更为详尽的答案，关于LLVM，可以参阅[这篇将于11月发布在OOPSLA的文章](http://www.cs.utah.edu/~regehr/oopsla18.pdf "Reconciling High-Level Optimizations and Low-Level Code
in LLVM"){% post_link 2018-09-29-Article-Translations-1 [以及我对这篇文章的翻译] %}，此外作者的一位合作者也写了[一篇关于这个材料的博客](https://www.ralfj.de/blog/2018/07/24/pointers-and-bytes.html "Pointers Are Complicated, or: What's in a Byte?"){% post_link 2018-09-27-Program-Tips-2 [以及我对这篇博客的翻译] %}