---
title: 文献翻译（一）：在LLVM中协调高级优化以及低级代码
date: 2018-09-29 10:24:14
tags: Article
categories: Article # 分类
thumbnail: /assets/img/posts/Article-Translations/default.jpg
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对国外文献做的个人翻译，并在此附上原文地址
>[Reconciling High-Level Optimizations and Low-Level Code in LLVM](http://www.cs.utah.edu/~regehr/oopsla18.pdf "Reconciling High-Level Optimizations and Low-Level Code in LLVM")


# 在LLVM中协调高级优化以及低级代码

LLVM会在Rust使用行指针或是C与C++中进行整型和指针的转换这种低级语言特性的时候会出现编译错误。问题在于对于编译器来说，在保证遵照编程语言对于低层次程序的规范上去实现有侵略性，高层次的内存优化是很困难的。一个更大的困难就是LLVM的内存模型的中间表示IR「intermediate representation」是非正式的并且编译器的开发者并不是总清楚角落案例的语义。

我们针对LLVM的IR开发了一种新颖的内存模型并将其形式化了。这个新的模型需要删除一些其中有问题的IR层级的优化，但其也同时支持一些以前是非法的新的优化方法。我们已经实现了这个新的模型，并且其修复了一些已知的依赖于内存模型的编译错误，且不影响最终生成的代码质量。

CCS 概念：软件及其工程 -> 语义学；编译器

额外的关键字及短语： IR内存模型，LLVM

ACM引用格式：
Juneyoung Lee, Chung-Kil Hur, Ralf Jung, Zhengyang Liu, John Regehr, and Nuno P. Lopes. 2018. Reconciling
High-Level Optimizations and Low-Level Code in LLVM. Proc. ACM Program. Lang. 2, OOPSLA, Article 125
(November 2018), 28 pages. https://doi.org/10.1145/3276495

# 1 简介

一个程序语言的内存模型定义了这门程序是被允许如何去观察和影响储存的。举个例子，在Java中的引用是类似于指针的，他们将对象特别地标记了出来。但程序并不被允许去从对象的中间以及从头去建立引用。总的来是，一个真的低级编程语言，比方说汇编，允许任意的内存空间被监测和操作，并且不论地址被怎样计算，上面的操作也不会受限。

C和C++都有一个有趣的壁龛，它们都倾向于去成为一门低级语言。系统语言——操作系统核心，虚拟机管理器，嵌入式固件以及编程语言运行——都倾向于被建立在一个或者其他的上面。为了制成这些应用，被指向对象的指针能通过指针算数来被创建，并且指针能转化为整数型，且整数型能被转化为指针。然而，除开他们的低级特性，C和C++的内存模型也同时合并了高级特性。比方说，编译器被允许去假设「有一些限制」一个指向以分配内存的对象的指针不会被用于创建指向另一个对象的指针的根据。C和C++标准委员会倾向于在必要时让语言去提供低级语言特性的内存模型，但与此同时，保证其在高级语言特性下的大量优化代码的能力。这种在低级与高级语言特性之间的紧张关系很明显，并给编译器以及应用程序的开发人员带来了很多问题。

这篇文章就是关于编译器中间表示「IRs」的顺序话内存模型。IR层很重要因为其是高级优化发生之地。GCC和LLVM都具有非正式指定的内存模型，其中包括了那些导致低级程序错误编译的端到端问题。[附录A](#appendix-a)显示了一个C语言程序，其同时GCC与LLVM均编译错误。然而，我们所攻击的问题远远不止C和C++：[附录B](#appendix-b)显示了一个低级「但是是安全的」Rust函数，其被LLVM错误的编译了。其罪魁祸首是我们在LLVM全局值编号「GVN」优化中所发现的新BUG。GVN由分支条件中传播指针「以及整型」的等价，使用值相等的去替换指针。这个会改变程序的行为，因为指针的相等并不是完全「必然」等同。这个编译错误我们追踪回去会发现第二个LLVM中的BUG——其错误的假设`(int*)(intptr_t)p`与p等价。

修复这些现在IR语义的编译错误是可行的，但会禁用掉一些有用的优化。我们的主要贡献，其建立在「Kang et al. 2015」[^8]上面，是一个新的，正式的LLVM的IR内存模型，且其与现今的设计有两点不同。首先，其使用延迟边界检查来放宽了对于越界指针创建的限制，以便正确地执行有用的代码优化，其次，它使用了双子内存分配，即指针的值必须被显式地观察到，它不能是被猜测的。双子内存分配支撑了基于LLVM的语言的低级语言代码特性的侵略性优化——比方说整型-指针转换。我们将LLVM进化到了新的语义下，以便显示其并不需要对编译器的大幅度改变，亦不会对生成的代码质量有所损失。

# 2 背景：IR的内存模型

在这个篇章里，我们描述为了低级语义内存模型的设及空间并接解释为什么其不足以管理低级模型和高级优化之间的紧张关系。然后在第3章里，我们描述了LLVM IR 的新的内存模型的基本，并在第4章立正式实现它。我们的例子写成C的形式以让其有较好的阅读性，即使我们本文的内容是编译器IR而不是C的指定内存模型。

## 2.1 平面内存模型

两个内存模型的主要问题：「1」一个读出的指令的实际返回值是什么？「2」什么情况下一个内存-获取指令是算被较好地定义了？一种回答是内存模型应该定义储存指令写入的内存位置。

举个例子：下面的代码会打出什么？或者说`p[6] = 0`会改变q指向的对象的任何值嘛。

```c
char *p = malloc(4);
char *q = malloc(4);
q[2] = 0;
p[6] = 1;
print(q[2]); // prints 0 or 1?
```

在一个平面内存模型中，这个程序会输出1，如果`q == p + 4`的话，其余情况会输出0。平面内存模型将指针看作整型，因此这个程序允许去猜测对象的位置，就像我们「图1」中所显示的。一些汇编语言有平面内存模型，而其他的比方说带有分段内存的机器，就没有。

平面内存模型的概念是如此简单并且对于低级编程是十分适合的。但其阻碍了高级优化，而这些高级优化是现代编译中必不可少的。

## 2.2 数据流的源入口追踪

在过去的例子中，我们已经给出了那个程序可以根据内存块放的位置得出0或者1。这对分配器的运行时行为的过度依赖的约束了编译器，阻止了其继续重要优化——比如储存转发。举例来说，我们期望编译器能够
将存储`q[2] = 0`指令传至打印指令。因此，内存模型需要一个方法在我们无视p与q的最终指向哪里时又需要获取`q[2]`的值时去阻止向`p[6]`传值。
比方说，这个效应的规则已经从C89开始就是C的一部分了。

数据流的源入口追踪提供了一个方法去阻止我们获取从一个无关对象所派生出的指针中获取对象。这个想法是每个指针都是一对值：其所指向的对象以及内存地址「或者说对象的偏移量」。获取这个对象的越界内存是一个未定义行为UB。这个语义就表明了编译器会得出`p[6]`是不能访问`q[2]`的，不管事实上在运行时它们会石佛偶会引用到同一片地址。

数据流的源入口追踪的定义如下：

```c
char *p = malloc(4); // (val=0x10, obj=p)
char *q = malloc(4); // (val=0x14, obj=q)
char *q2 = q + 2; // (val=0x16, obj=q)
char *p6 = p + 6; // (val=0x16, obj=p)

*q2 = 0; // OK
*p6 = 1; // UB, since out-of-bounds of obj p
print(*q2); // can be replaced with print(0);
```

第一个通过`q2`的储存成功了，因为其在对象q的边界内。第二个，触发了UB，因为这个指针基于p，其越界了，及时程序成功的得到了有效对象的地址「如图二所显示的」。最终，因为这里在他们之间没有有效定义的存储指令，编译器能够安全地将存储`*q2 = 0`的指令传到打印指令中。

## 拓宽整型的来源

在上一节我们所表现出的并不支持低级语言特性——比方说整型-指针转换。为了支持这个功能，我们可以拓展来源信息：

```c

```











# 3


# 4



























# Appendix-A

附录A code
       
# Appendix-B

附录B code

# 参考文献

[^1]:Frédéric Besson, Sandrine Blazy, and Pierre Wilke. 2014. A Precise and Abstract Memory Model for C Using Symbolic Values. In APLAS. https://doi.org/10.1007/978-3-319-12736-1_24
[^2]:Frédéric Besson, Sandrine Blazy, and Pierre Wilke. 2015. A Concrete Memory Model for CompCert. In ITP. https://doi.org/10.1007/978-3-319-22102-1_5
[^3]:Frédéric Besson, Sandrine Blazy, and Pierre Wilke. 2017a. CompCertS: A Memory-Aware Verified C Compiler Using Pointer as Integer Semantics. In ITP. https://doi.org/10.1007/978-3-319-66107-0_6
[^4]:Frédéric Besson, Sandrine Blazy, and Pierre Wilke. 2017b. A Verified CompCert Front-End for a Memory Model Supporting Pointer Arithmetic and Uninitialised Data. Journal of Automated Reasoning (03 Nov 2017). https://doi.org/10.1007/s10817-017-9439-z
[^5]:David Chisnall, Justus Matthiesen, Kayvan Memarian, Peter Sewell, and Robert N. M. Watson. 2016. C memory object and value semantics: the space of de facto and ISO standards. https://www.cl.cam.ac.uk/~pes20/cerberus/notes30.pdf
[^6]:Arie Gurfinkel and Jorge A. Navas. 2017. A Context-Sensitive Memory Model for Verification of C/C++ Programs. In SAS. https://doi.org/10.1007/978-3-319-66706-5_8
[^7]:Chris Hathhorn, Chucky Ellison, and Grigore Roşu. 2015. Defining the Undefinedness of C. In PLDI. https://doi.org/10.1145/2737924.2737979
[^8]:Jeehoon Kang, Chung-Kil Hur, William Mansky, Dmitri Garbuzov, Steve Zdancewic, and Viktor Vafeiadis. 2015. A Formal C Memory Model Supporting Integer-pointer Casts. In PLDI. https://doi.org/10.1145/2737924.2738005
[^9]:Robbert Krebbers. 2013. Aliasing Restrictions of C11 Formalized in Coq. In CPP. https://doi.org/10.1007/978-3-319-03545-1_4
[^10]:Robbert Krebbers and Freek Wiedijk. 2015. A Typed C11 Semantics for Interactive Theorem Proving. In CPP. 15–27. https://doi.org/10.1145/2676724.2693571
[^11]:Juneyoung Lee, Yoonseung Kim, Youngju Song, Chung-Kil Hur, Sanjoy Das, David Majnemer, John Regehr, and Nuno P.Lopes. 2017. Taming Undefined Behavior in LLVM. In PLDI. https://doi.org/10.1145/3062341.3062343
[^12]:Xavier Leroy. 2009. Formal Verification of a Realistic Compiler. Commun. ACM 52, 7 (July 2009), 107–115. https://doi.org/10.1145/1538788.1538814
[^13]:Xavier Leroy and Sandrine Blazy. 2008. Formal Verification of a C-like Memory Model and Its Uses for Verifying Program Transformations. Journal of Automated Reasoning 41, 1 (Jul 2008), 1–31. https://doi.org/10.1007/s10817-008-9099-0
[^14]:Nuno P. Lopes, David Menendez, Santosh Nagarakatte, and John Regehr. 2015. Provably Correct Peephole Optimizations with Alive. In PLDI. https://doi.org/10.1145/2737924.2737965
[^15]:Kayvan Memarian, Justus Matthiesen, James Lingard, Kyndylan Nienhuis, David Chisnall, Robert N.M. Watson, and Peter Sewell. 2016. Into the depths of C: elaborating the de facto standards. In PLDI. https://doi.org/10.1145/2908080.2908081
[^16]:Kayvan Memarian and Peter Sewell. 2016a. Clarifying the C memory object model (revised version of WG14 N2012). https://www.cl.cam.ac.uk/~pes20/cerberus/notes64-wg14.html
[^17]:Kayvan Memarian and Peter Sewell. 2016b. N2090: Clarifying Pointer Provenance (Draft Defect Report or Proposal for C2x). https://www.cl.cam.ac.uk/~pes20/cerberus/n2090.html
[^18]:The LLVM Project. 2018. LLVM Language Reference Manual. https://llvm.org/docs/LangRef.html
[^19]:Raimondas Sasnauskas, Yang Chen, Peter Collingbourne, Jeroen Ketema, Jubi Taneja, and John Regehr. 2017. Souper: A Synthesizing Superoptimizer. CoRR abs/1711.04422 (2017). http://arxiv.org/abs/1711.04422
[^20]:Jaroslav Ševčík, Viktor Vafeiadis, Francesco Zappa Nardelli, Suresh Jagannathan, and Peter Sewell. 2013. CompCertTSO: A Verified Compiler for Relaxed-Memory Concurrency. J. ACM 60, 3, Article 22 (June 2013), 22:1–22:50 pages. https://doi.org/10.1145/2487241.2487248
[^21]:Wei Wang, Clark Barrett, and Thomas Wies. 2017. Partitioned Memory Models for Program Analysis. In VMCAI. https://doi.org/10.1007/978-3-319-52234-0_29
[^22]:Jianzhou Zhao, Santosh Nagarakatte, Milo M.K. Martin, and Steve Zdancewic. 2012. Formalizing the LLVM Intermediate Representation for Verified Program Transformations. In POPL. https://doi.org/10.1145/2103656.2103709














