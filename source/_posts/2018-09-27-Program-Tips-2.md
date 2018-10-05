---
title: 博文翻译（二）：指针很复杂，或者：一个字节中有什么
date: 2018-09-27 21:54:36
tags: Program
categories: Program # 分类
thumbnail: /assets/img/posts/Program-Tips/1.jpg
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对国外博客做的个人翻译，并在此附上原文地址
>[Pointers Are Complicated, or: What's in a Byte?](https://www.ralfj.de/blog/2018/07/24/pointers-and-bytes.html "ralfj.de")

这整个2018年的暑假，我都在[不停地着力于Rust](https://www.ralfj.de/blog/2018/07/11/research-assistant.html "Back at Mozilla")，并且再一次我会工作于（除开其他事项外）在Rust/MIR的 __内存模型__。然而，在谈及我这一年有的新想法之前，我最后必须花些时间去破除这个“指针很简单：它们仅仅是整型类型”这个神话了。这个表述的两个方面都是错的，至少对于那些使用了不安全的特性的语言，比方说Rust或者C：指针不仅不简单，它也不(仅仅)是整型的。

我仍旧期望在我们讨论更加复杂的部分之前，能够定义一段必须被修复的内存模型：被储存在内存中的数据到底是什么？它以字节「Bytes」、最小可寻址单元以及（在大多数平台上的）可访问最小内存段组成。但字节的可能是是多少？再一次重申，“这仅仅是一个8字节整型数”并不能被我们当作答案。

我期望在这篇文章的最后，你能够同意我关于这些的陈述。 :)

# 指针是复杂的

“指针就仅仅是整型”这个说法的错在哪儿？让我们来看看下面这个例子：

「我在下面一般会使用`C++`代码因为在`C++`中写不安全代码比在`Rust`中容易得多，并且非安全代码就是那些问题出现的地方。`C`也有所有这些同样的问题，使用`Unfafe Mode`的`Rust`代码也是」

```language-c
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

第一条规定，不允许执行指针运算「比方说像&x[i]做得那样」[超出其开始数组的任意一端](https://timsong-cpp.github.io/cppwp/n4140/expr.add#5 "Additive operators")。我们的程序违反了这条规则：`x[i]`在`x`之外，所以这就是一个未定义行为UB。我们更清楚的解释一下：仅仅是`x_ptr`的计算就已经是UB了，我们甚至都没有到我们使用这个指针的地方[^1]。

但我们仍旧没有结束：这条规则有一个特殊的例外能让我们利用我们的优势。如果算数最终结束计算指针的得到的结果刚好在一个内存分配块的结尾，那个计算就是无害的「这条例外十分重要，其允许在`C++`98版本的迭代循环中计算`vec.end()`」。

>`vec.end()`：回传一个Iterator，它指向 vector 最尾端元素的下一个位置（请注意：它不是最末元素）

所以我们对上面的例子做一点小小的改变：

```language-c
int test(){
    auto x = new int[8];
    auto y = new int[8];
    y[0] = 42;
    auto x_ptr = x+8;
    if (x_ptr == &y[0]){
        *x_ptr = 23;
    }
    return y[0];
}
```

现在，去试想一下`x`和`y`都被分配到了相邻的内存块上，并且`y`的地址比`x`的大。明显的，`x_ptr`实际上指向了`y`的地址开头，我们`if`语句的条件判断为真并进行了赋值操作。但是由于我们使用了越界指针算法，这里没有未定义行为UB。

这个看上去似乎破坏了优化。然而我们`C++`标准有另外一个技巧去帮助编译器作者：并不是真正允许我们去使用我们上面的`x_ptr`。根据`C++`标准，它提到了[指针的加法规则](https://timsong-cpp.github.io/cppwp/n4140/expr.add#5 "Additive operators")，`x_ptr`指向了“超过了一个数组对象的最后一个元素”。即是他们有相同的地址，但仍旧不会指向另外一个实际对象的元素。「至少，那就是[LLVM标准在优化这段代码时](https://godbolt.org/z/XY3ecn "Compiler Explorer")的通用解释」

在这里面的关键点就是因为`x_ptr`和`&y[0]`指向了同一个地址，那并不会让他们变成同一个指针。它们并不能互换使用：`&y[0]`指向了`y`的第一个元素，而`x_ptr`指向了`x`的最。如果我们用`*&y[0] = 0`代替`*x_ptr = 23`，我们会改变这程序的含义，即使两个指针已经被测试过等价性。

这值得重复一遍：
**仅仅因为两个指针指向了同一个地址，并不意味着他们能够相互替代甚至是相等！**

这似乎看起来很微妙，确实是。事实上，这仍旧导致了[LLVM](https://bugs.llvm.org/show_bug.cgi?id=35229 "Bug 35229 - LLVM Memory Model needs more rigor to avoid undesired optimization results")与[GCC](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=65752 "Bug 65752 - Too strong optimizations int -> pointer casts")中编译时的差异。

我们同时注意到了这个末端原则并不是`C++`中唯一一个可以见证这个现象的地方。我们看另外一个例子：`C`中的`restrict`关键字，这个能被用来表示指针没有别名。

```language-c
int foo(int *restrict x, int *restrict y){
    *x = 42;
    if (x == y){
        *y == 23
    }
    return *x;
}

int test(){
    int x;
    return foo(&x, &y)
}
```

当运行`test()`就会触发UB，因为在`foo()`中的两个来源都必须没有别名。在`foo()`中通过`*x`替换`*y`的值已经改变了程序的意义让这个不再UB。所以再一次重申，即使`x`和`y`都有相同的地址，他们也不能相互替换。

指针确实不是整型类型。


# 一个简单的指针模型

那么，到底指针是什么？我并不清楚真正的答案。事实上，这是一片开放的研究领域。

我们在这要强调一点就是我们仅仅在探究指针的抽象模型。当然，在实际的机器码里，指针是整数型。但实际机器码并不会做像现代`C++`编译器做得那些优化工作，所有我们可以把这个抛到一边。如果我们在汇编语言里面写下上面的程序，这里就不会有UB，也没有优化。`C++`和`Rust`有一个关于内存和指针的更“高维度”的视角，限制住了编程人员并获得优化。当正式地描述了编程人员在这些语言中该做和不该做的事情后，正如我们所看见的，将之战看作整数型的模型就不管用了，我们得找到一个其他的模型代替它。这就是另外一个例子，其使用了不同于真实机器的“虚拟机”用于规范的目的，这其中的想法我在[之前的博客中已经有所讨论](https://www.ralfj.de/blog/2017/06/06/MIR-semantics.html "xploring MIR Semantics through miri")

这里有一个简单的方案「事实上，这也是我们在[CompCert](https://hal.inria.fr/hal-00703441/document "The CompCert Memory Model, Version 2")以及我那个[RustBelt work](https://www.ralfj.de/blog/2017/07/08/rustbelt.html "RustBelt: Securing the Foundations of the Rust Programming Language")中的指针模型。在这里你也可以看见[miri](https://github.com/solson/miri/ "An interpreter for Rust's mid-level intermediate representation")是怎么[实现指针](https://github.com/rust-lang/rust/blob/fefe81605d6111faa8dbb3635ab2c51d59de740a/src/librustc/mir/interpret/mod.rs#L121-L124 "rust/src/librustc/mir/interpret/mod.rs")的」:一个指针就是一对拥有唯一标识内存分配的ID，以及对于这个内存分配的偏移量。如果我们在RUST中定义它，我们可能会写成：

```rust
struct Pointer {
    alloc_id: usize,
    offset: isize
}
```

从/向一个指针中加/减一个整数型仅仅体现在偏离量上，并且因此不会离开其内存分配区。只有当这两个指针同时指向了同一个内存分配区「[C++中的体现](https://timsong-cpp.github.io/cppwp/n4140/expr.add#6 "Additive operators")]」时，才允许两个指针之间的相减。[^2]

这最终变成「以及miri所表现出来的」这样了，这个模型能让我们获益良多。我们永远会记住我们指针最开始所指向的内存分配区，这样我们就能将那种内存分配上的“结尾过一”「one-past-the-end」型指针与另外一块内存分配开端的指针区别开来。那就是为什么miri能够检查出我们第二个例子「关于`&x[8]`」是未定义行为UB。

# 模型分崩离析

在这个模型里，指针并非整型数，但它们至少还是很简单的。然而一旦你开始思考指针到整数型的转换了，这整个模型就开始崩解。在miri中，将一个指针转换为整数型其实并没有做什么事情，我们现在仅仅是拥有一个整数型变量「即，它的类型声称其是一个整数型」，但他的值是一个指针「一对内存分配-偏移量」。然而，将那个“整数”乘以2会导致一个错误，因为这事实上并不清楚把一个实际的指针乘以2有什么意义。

我必须先澄清一下，当定义语言的语义时这并不是一个好的解决办法。虽然这在解释器「interpreter」上工作时表现的很不错。这是一个最懒于做的事情，并且我们做这个是因为不清楚还有什么其他的可以做「除了一点都不支持这种类型转换操作，但有这个，miri能够运行更多的程序」：在我们的抽象机器上，这里没有单独的连续“地址空间”供以内存的分配，因此我们可以用映射表「map」来将每一个指针映射到一个整数型上。每个内存分区都仅仅被一个「不可被观测的」ID所定义或标记。我们现在可以开始使用二外的数据来丰富这个模型，比如说每个内存分配的基础地址，并在我们将整数型转换为指针的时候使用它……但那就说的太过于复杂了，并且讨论那种模型也不是本文的重点。重点是讨论这种模型的__需要__。如果你对此感兴趣，我推荐你去阅读[这篇文章](http://www.cis.upenn.edu/~stevez/papers/KHM+15.pdf "A Formal C Memory Model
Supporting Integer-Pointer Casts")，其探讨了添加一个基础地址相关的问题。

长话短说，指针到整数型的转换操作在我们同时考虑我们上面所提到的优化问题时是混乱且嫩一被正式定义的。在需要允许优化的高层级与需要解释指针到整数型转换得到低层级之间是冲突的。我们在miri中一般就简单地忽略了这个问题并尽可能的做我们能做的——给出一个我们现在着力于的简单模型。像`C`、`C++`或者`Rust`中的完整定义当然不可能这么简单。必须去解释到底这里真正发生了什么。在我的只是范围内，这问题没有一个完美的解法，但学术研究[已经接近了](https://sf.snu.ac.kr/publications/llvmtwin.pdf "Reconciling High-Level Optimizations and Low-Level Code
in LLVM")。

>当然这绝不是一般性的`C`语言的学术研究的详尽清单。我不知道其他关于整数-指针转换以及优化的相关工作，但其他关于`C`的相关工作我们可以看看[KCC](https://github.com/kframework/c-semantics "Semantics of C in K")、[Robbert Krebber 的博士论文](https://robbertkrebbers.nl/thesis.html "The C standard formalized in Coq")以及[Cerberus](https://www.cl.cam.ac.uk/~pes20/cerberus/ "Cerberus")

上面又再次解释了为什么指针并不简单。


# 从指针到字节

我希望我已经做出了足够有说服力的论证来证明整数型并不是唯一需要被考虑的数据类型当我们正式指定低级语言比方说`C++`或者「使用不安全模式部分的」`Rust`。然而，他和意味着一个简单的操作比方说从内存读取一个字节不能简单地返回一个`u8`「8-bit无符号整型」。试想一想，我们通过将源的每个字节「按顺序」加载进某个局部变量`v`中，再将其储存到目标中[实现`memcpy`函数](https://github.com/alexcrichton/rlibc/blob/defb486e765846417a8e73329e8c5196f1dca49a/src/lib.rs#L39 "rlibc/src/lib.rs")。当它的第一个字节是指针的一部分该怎么办？我们不得不找到到底`v`的值是什么，所以我们必须找到一些方法去回答这个问题。「这与上一节中所出现的乘法问题是完全不同的。我们仅仅假设一些抽象类的指针」

我们并不能将一个指针的1字节看作一个`0..256`的元素。实际上，如果我们使用一个内存的简单模型，指针的多余“隐藏”数据「用来标记其与整数型不同」将会在我们将一个指针储存到内存再读出后丢失。我们得解决这个问题，所以我们必须得拓展我们“字节”的标记去适应这种额外情况。所以，现在一个字节要不就是一个`0..256`「raw bits」元素，亦或是某些抽象指针的第n比特。如果我们在`Rust`中去实现我们的内存模型，就看起来像下面这样：

```rust
    enum ByteV1{
        Bits(u8),
        PtrFragment(Pointer, u8),
    }
```

举个例子，`PtrFragment(ptr, 0)`表示`ptr`的第一个字节，通过这种方式，`memcpy`可以"分离"一个指针，将其变成独立的可以在内存中表示这个指针的字节，并分别地复制它们。在一个32位体系结构中，`ptr`的完整值由下面4个字节组成：

```language-c
[PtrFragment(ptr, 0), PtrFragment(ptr, 1), PtrFragment(ptr, 2), PtrFragment(ptr, 3)]
```
这种表示支持对指针进行的所有字节级别的“数据移动”操作，这对于`memcpy`是足够的。但不完全支持算数或者位(bit)级别的操作：正如上面所说的，那需要更为复杂的指针表示。


# 未初始化的内存

然而，我们仍旧没有完成我们对于“字节”的定义。为了完全描述程序行为，我们需要考虑更多的可能性：一个在内存中的字节有可能并没有被初始化。最终的对于字节的定义「假定我们有一个`Pointer`类来定义指针」会像下面一样：

```rust
    enum Byte {
        Bits(u8),
        PtrFragment(Pointer, u8),
        Uninit
    }
```

其中`Uninit`是我们对所有我们已经分配了内存但仍没有对其写入的字节所设置的属性。读取未初始化的内存没有问题，但这些字节做任何实际的操作「比如把它们用在整数运算上」都是一个未定义行为UB。

这个和`LLVM`对它叫`poison`的特别值所做得规则很相似。注意`LLVM`同时也有一个叫`undef`的值，`Rust`将其用在未初始化内存上并且其工作原理也有所不同——然而，将`Uninit`编译为`Undef`实际上是正确的「`Undef`在某种意义上表示“较弱”」，并且这里有些声音建议[从LLVM中移除`Undef`并使用`poison`代替]。

你可能想知道为什么我们有一个特别的`Uninit`值。难道我们不能仅仅是为每个新分配内存的字节去选择任意的一个`b:u8`，再使用`Bits(u8)`当成其初始化值？那确实也能当作一个选项。然而，第一，相当多的编译器都已经融合到了对于未初始化的内存具有标记值。没有做那个将不仅仅导致`LLVM`编译时出现错误，它还需要重新评估许多优化，以确保它们能够正常工作于这次要更改的模型。关键点就是在编译过程中，用任何值去替换`Uninit`总是安全的：任何实际观察该值的操作都是未定义操作UB。

举个例子，下面的`C`代码使用`Uninit`会更加容易优化：

```language-c
int test(){
    int x;
    if (condA()) x = 1;
    //许多难以分析的代码会在 condA()成立时返回
    //不会持有，但也不会改变x
    use(x) //希望去优化x到1
}
```

有了`Uninit`，我们能够轻松地争论`x`的值到底是`Uninit`或者`1`，并且因为使用`1`去取代`Uninit`是允许的，优化过程能轻易地判断。如果没有`Uninit`，`x`的值就可能是“其它任意位的格式”或者`1`，这样做同样的优化的判断时就会变得相当困难。[^3]

最后，`Uninit`也同时是像miri一样的解释器的一个更好的选择。那些解释器花了很麻烦的一段时间去解决那些像“就去选这些值中的任意一个”「比方说上面的不确定性操作」这种形式的操作，因为如果它们想要完全的去搜索所有的程序结果会意味着他们得去尝试所有的可能值。使用`Uninit`而不是一个任意的位「bit」数据意味着miri可以在一次执行后就可靠的告诉你程序是否错误的使用了为初始化的值。

# 结论

我们已经看了那些在譬如`C++`和`Rust`「不像真正的硬件」语言中，指针们即使在指向相同地址时也有所差异。并且一个字节也远不止一个在`0..256`中的数字。这也是为什么在1978年调用`C`“便携式装配”可能是合适的，但在现在是一个危险的误导性陈述。有了这个，我觉得我们已经准备好去下一篇文章中查看我的“2018内存模型「还在写标题」”的初稿了。:)

感谢 @rkruppe 和 @nagisa 的帮助去找到为什么`Uninit`是需要的论据。如果你对此有什么疑问，在[这个表单里面可以自由的询问我](https://internals.rust-lang.org/t/pointers-are-complicated-or-whats-in-a-byte/8045/4 "Pointers Are Complicated, or: What's in a Byte")！



# 脚注

[^1]: 这最终会导致 `i = y-x` 也是未定义行为UB。因为[一个可能是仅仅将指针减到相同的内存分配中](https://timsong-cpp.github.io/cppwp/n4140/expr.add#6 "Additive operators")，然而我们可以使用`i = ((size_t)y - (size_t)x)/size_t(int)`来解决那问题。

[^2]: 正如我们所看见的，`C++`标准实际上将这条规则运用到了数组的层级上，而不是内存分配上。然而，`LLVM`将这条规则运用到了 [每一个分配级别](https://llvm.org/docs/LangRef.html#getelementptr-instruction "LLVM Language Reference Manual - getelementptr" )上。

[^3]: 我们可以争辩说，当做出费确定性选择时我们可以重新排序，但我们随后需要证明这些难以分析的代码并没有观测到 `x`。`Uninit`避免了那些不必要的额外证明负担。