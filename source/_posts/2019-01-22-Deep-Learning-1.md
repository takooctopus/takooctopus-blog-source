---
title: DeepLearning（一）：简介
date: 2019-01-22 17:44:23
tags: DeepLearning
categories: ArtificialIntelligence # 分类
thumbnail: /assets/img/posts/ArtificialIntelligence/DeepLearning.jpg
---

# 简介

为了之后的毕设，不得不回来看看这本之前没看的下去的书，当然，当年是中文版的，现在是英文版的。

之前看不下去还有很大部分是因为数学功底的问题，时隔这么久，还是真正去做而不是粗略的看才更有效一些。

整本书大致分为了20个章节「这就是个目录，会随着更新慢慢添加链接的」：

- Introduction 简介
- Linear Algebra 线性代数
- Probability and Information Theory 概率论和信息论
- Numerical Computation 数值分析
- Machine Learning Basics 机器学习基础
- Deep Feedforward Networks 深度前馈网络
- Regularization for Deep Learning 深度学习的一般化
- Optimization for Training Deep Models 训练模型的最优化
- Convolutional Networks 卷积神经网络
- Sequence Modeling: Recurrent and Recursive Nets 序列建模：循环和回归网络
- Practical Methodology 实践设计流程
- Application 应用
- Linear Factor Models 线性因子模型
- Autoencoders 自编码器
- Representation Learning 代表模型
- Structured Probabilistic Models for Deep Learning 结构化多样模型：深度学习的模型
- Monte Carlo Methods 蒙特卡罗算法
- Confronting the Partition Function 面对配分函数
- Approximate Inference 近似
- Deep Generative Models 深度生成模型

## 符号

关于在看整本书的过程中，我们还是需要去确认整个符号的使用

得了 :(

看了半天不晓得有几个咋打出来的

我就干脆直接放PDF吧

{% asset_img notation-1.jpg %}
{% asset_img notation-2.jpg %}
{% asset_img notation-3.jpg %}
{% asset_img notation-4.jpg %}


## 影响因素

- 表示方式
- 特征的提取
    - 一种解决方法是：用深度学习不仅去寻找特征的到结果的映射而是去提取`特征`本身。「学习到的特征相比我们自己做的特征(hand-crafted)一般性能更好，还更省时间和人力」
    - 典型应用就是：自动编码器(autoencoder)
    - 目的是分离出那些能解释观测数据的变量的`因素(factor)`「这里的因素并非能够直接观测到的，而是存在于那些现实中无法观测的物件或者力里的」就把它想成那些让数据中的丰富变量能够有意义的概念或者抽象就行。
    - 这种高层次、抽象的特征是很难提取的，很多这种随机因素只能用那种特别复杂的，人类层面的理解才能确认出来。

### 深度学习

深度学习就解决了上面那个核心问题——它能够对复杂的概念建模。

下面的图就显示了一个深度学习系统是如何表示一张人的图片概念的：

{% asset_img figure-1-2.jpg %}

<label class="label-example">fig 1.2 深度学习的模型图</label>

我们可以看见最下层是原始感官输入数据「由于它们是由像素值组成的，我们很难里就这个的含义」深度学习会将这个分解成为一系列嵌套的映射，每个映射都由模型的一层去描述。

输入由可视层描述，然后是一系列的隐藏层「他们被称为隐藏，因为原始数据中没有」

典型的例子就是前馈神经网络或者多层感知器(MLP)。一个多层感知器就是一个数值函数将输入集去映射到输出集，整个就是由许多简单函数组成的。

另外一个前景就是其能允许电脑去学习多步的电脑程序，每一个表示层都能被视作电脑内存在并行运行完另外一个指令集合后的状态。拥有更深的网络可以在序列中执行更多的指令。序列指令会提供更好的效果「因为后面的指令可以回馈之前指令的结果」，即我们的表示(representation)不仅编码了解释输入的变量因素，同时也储存了那些帮助输入信息去产生实际意义的信息。「他们可能类似于传统程序中的计数器(counter)或者指针(pointer)起到的作用」这能够帮助我们的模型去组织它的处理层次。

这里有两种主要方式去测量`模型的深度`:

- 第一种基于那些`必须被运行的序列指令的数目`，并借此去去评价整个结构。这个就是整个流图中描述从输入到输出的最长路径「因此同样程序用不同语言写出来会有所不同」，下图会有关于这个的示意图：

{% asset_img figure-1-3.jpg %}

<label class="label-example">fig 1.3 </label>

我们可以看到上图左图中输出是`逻辑回归模型(logistic regression model)`，即 $ \sigma(w^Tx) $ ，这样我们这个程序就有三个步骤了「相乘，相加和逻辑回归」即层深为3。而右图中将整个逻辑回归视作一个元素，那么层深只有1。

- 另外一种方法，将整个模型的深度视作`概念是如何和其他的概念相关联的`而非是计算图的深度。这样出来的流图深度可能远远大于概念本身的深度。「简单的概念可能被系统精制成为一个更为复杂的概念」

{% asset_img figure-1-4.jpg %}

<label class="label-example">fig 1.4 各个AI学科之间的关系</label>

{% asset_img figure-1-5.jpg %}

<label class="label-example">fig 1.5 每个工作原理的详细示意图</label>

## 学习流程图

为此，我们提供了整个流程图，以让大家去选择其学习计划：

{% asset_img flowchart.jpg %}


## 单词

arithmetic 算数
Cartesian 笛卡尔
disentangle 解开
sophisticated 复杂的
quintessential 典型的
analogous 类似
hierarchy 等级制度
disciplines 学科
rebrand 更名、贴牌
cybernetics 控制论
connectionism 联结
artificial neural networks (ANNs)
adaptive linear element (ADALINE)
stochastic gradient descent 随机梯度下降
fragmented 支离破碎
Neocognitron 神经认知机
rectified linear unit 整流线性单元
cognitive science 认知科学
Biological neurons 生物神经元
leech 水蛭

