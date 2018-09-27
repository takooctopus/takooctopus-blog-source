---
title: 博文翻译（二）：指针很复杂，或者：一个比特中有什么
date: 2018-09-27 21:54:36
tags: Program
categories: Program # 分类
thumbnail: /assets/img/posts/Program-Tips/1.jpg
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对国外博客做的个人翻译，并在此附上原文地址
>[Pointers Are Complicated, or: What's in a Byte?](https://www.ralfj.de/blog/2018/07/24/pointers-and-bytes.html "ralfj.de")

这整个2018年的暑假，我都在[不停地着力于Rust](https://www.ralfj.de/blog/2018/07/11/research-assistant.html "Back at Mozilla")，并且再一次我会工作于（除开其他事项外）在Rust/MIR的 __内存模型__。然而，在谈及我这一年有的新想法之前，我最后必须花些时间去破除这个“指针很简单：它们仅仅是整型类型”这个神话了。这个表述的两个方面都是错的，至少对于那些使用了不安全的特性的语言，比方说Rust或者C：指针不仅不简单，它也不(仅仅)是整型的。

我仍旧期望在我们讨论更加复杂的部分之前，能够定义一段必须被修复的内存模型：被储存在内存中的数据到底是什么？它以字节「Bytes」、最小可寻址单元以及（在大多数平台上的）可访问最小内存段组成。但字节的可能是是多少？再一次重申，“这仅仅是一个8比特整型数”并不能被我们当作答案。

我期望在这篇文章的最后，你能够同意我关于这些的陈述。 :)

# 指针是复杂的

“指针就仅仅是整型”这个说法的错在哪儿？让我们来看看下面这个例子：

「我在下面一般会使用`C++`代码因为在`C++`中写不安全代码比在`Rust`中容易得多，并且非安全代码就是那些问题出现的地方。`C`也有所有这些同样的问题，使用`Unfafe Mode`的`Rust`代码也是」

```c
int test() {
    auto x = new int[8];
    auto y = new int[8];
    y[0] = 42
    int i = /* 一些没有副作用的计算「some side-effect-free computation」 */;
    auto x_ptr = &x[i];
    *x_ptr = 23;
    return y[0];
}
```

能够去优化最后`y[0]`的读数并仅仅返回`42`是极好的。这种优化的理由就是我们的代码仅仅向`x_ptr`中写入了，而它指向了`x`，并不能改变`y`的值。

然而，给出一个像`C++`一样的低级上面，我们能够事实上地打破这种假设，仅仅需要将`i`的值设置成`y-x`，因为`&x[i]`「即x[i]的地址」与`x+i`是一样的，这就意味着我们事实上将`23`写进了`&y[0]`中。

当然，那个并不会阻止`C++`编译器去继续他的优化。为了允许这个，`C++`标准声称我们的代码有未定义行为UB[「Undefined behavior」](https://www.ralfj.de/blog/2017/07/14/undefined-behavior.html "Undefined Behavior and Unsafe Code Guidelines")。

第一条规定，不允许执行指针运算「比方说像&x[i]做得那样」[超出其开始数组的任意一端](https://timsong-cpp.github.io/cppwp/n4140/expr.add#5 "Additive operators")。我们的程序违反了这条规则：`x[i]`在`x`之外，所以这就是一个未定义行为UB。我们更清楚的解释一下：仅仅是`x_ptr`的计算就已经是UB了，我们甚至都没有到我们使用这个指针的地方。[1](#fn:1)











# 脚注


