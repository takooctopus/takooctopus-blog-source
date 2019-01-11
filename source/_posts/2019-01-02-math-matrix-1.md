---
title: 矩阵分析与应用（一）：矩阵的基本运算
date: 2019-01-02 21:55:08
tags: Matrix
categories: Math # 分类
thumbnail: /assets/img/posts/Math/matrix.jpg
---

# 矩阵和线性方程组

## 矩阵的基本性质

### 矩阵和向量

我们看待`m x n`的线性方程组：

<p>
    $$
    \left.
    \begin{array}{l}
    a_{11}x_1 + a_{12}x_2 + ... + a_{1n}x_n = b_1 \\
    a_{21}x_1 + a_{22}x_2 + ... + a_{2n}x_n = b_2 \\
    ... \\
    a_{m1}x_1 + a_{m2}x_2 + ... + a_{mn}x_n = b_m \\
    \end{array}
    \right\}
    =f(n)
    $$
</p>

这里面有`m`个方程去描述了`n`个变量的线性关系，我们可以使用矩阵很容易的去标识出来：
<p>$$ Ax = b $$</p>

明显的，在上式中：
<p>
$$
    A = 
    \begin{bmatrix}
    a_{11}&a_{12}&\cdots&a_{1n}\\
    a_{21}&a_{22}&\cdots&a_{2n}\\
    \vdots&\vdots&&\vdots\\
    a_{m1}&a_{m2}&\cdots&a_{mn}\\
    \end{bmatrix}
$$
</p>

A是一个`m x n`阶矩阵

而x和b分别是`n x 1`和`m x 1`阶列向量

对于矩阵A，我们可以用线性系统、滤波器、无线信道等符号表示。

#### 向量

而对于向量，我们有下面的分类：

<p>
$$
    \begin{cases}
    \text{物理向量}\\
    \text{代数向量} 
        \begin{cases} 
            \text{常数向量} \\
            \text{函数向量} \\
            \text{随机向量} \\
        \end{cases}\\
    \text{几何向量}\\
    \end{cases}
$$
</p>

我们可以将整个大A矩阵分成若干个列向量的组合，即：

<p>
$$
    a_1 = 
    \begin{bmatrix}
    a_{11}\\
    a_{21}\\
    \vdots\\
    a_{m1}\\
    \end{bmatrix},
    a_2 = 
    \begin{bmatrix}
    a_{12}\\
    a_{22}\\
    \vdots\\
    a_{m2}\\
    \end{bmatrix},
    \cdots ,
    a_n = 
    \begin{bmatrix}
    a_{1n}\\
    a_{2n}\\
    \vdots\\
    a_{mn}\\
    \end{bmatrix}
$$
</p>

即我们可以用列向量表示A：
<p> $$ A = [a_1,a_2,\cdots,a_n] $$ </p>

#### 对角线

主对角线、次对角线的概念不用多说。

我们将除了主对角线以外的元素全为`0`的n阶矩阵记作对角矩阵，即
<p>$$ D = diag(d_1,d_2,\cdots,d_n) $$</p>

此外，如果对角矩阵主对角线上元素全为1，则称其为单位矩阵 $ I_{n \times n} $ ，如果全为0，则称为零矩阵 $ O_{n \times n} $

#### 子矩阵和分块矩阵

在这系列的博文中，我们会使用 $ A(i_1:i_p,j_1:j_q) $ 来代表由A矩阵中 $ i_1 \text{到} i_p $ 行，以及 $ j_1 \text{到} j_q $ 列的子矩阵。

而分块矩阵就是以矩阵作为元素的矩阵：

<p>
$$
    A = [A_{ij}] = 
    \begin{bmatrix}
    A_{11}&A_{12}&\cdots&A_{1n}\\
    A_{21}&A_{22}&\cdots&A_{2n}\\
    \vdots&\vdots&&\vdots\\
    A_{m1}&A_{m2}&\cdots&A_{mn}\\
    \end{bmatrix}
$$
</p>

## 矩阵的基本运算

我们用上代数的习惯：以R代表实数集合，C表示复数集合，我们可以定义复数矩阵A：

<p>
$$
A \in C^{m \times n}  \Leftrightarrow A = [a_{ij}], a_{ij} \in C, \qquad i = 1,2,\cdots,m; \  j=1,2,\cdots,n 
$$
</p>

同样的，实矩阵也能以相似的方式表示出来。

### 矩阵的转置、复共轭和复共轭转置「Hermitian」

首先是转置，我们设定 $ A = [a_{ij}] $ 是一个m x n阶矩阵，则我们记
<p>
$$
    [A^T]_{ij} = a_{ji} 
$$
</p>

为A的`转置`，记作 $ A^T $

而对于复矩阵，我们先去看它的复共轭矩阵  $ [A^ {\ast} ]_{ij} = a^ {\ast} _{ij} $

而`复共轭转置`我们定义为 $ A^H $

<p>
$$
A^H =  
    \begin{bmatrix}
    a_{11}^{\ast}&a_{21}^{\ast}&\cdots&a_{m1}^{\ast}\\
    a_{12}^{\ast}&a_{22}^{\ast}&\cdots&a_{m2}^{\ast}\\
    \vdots&\vdots&&\vdots\\
    a_{1n}^{\ast}&a_{2n}^{\ast}&\cdots&a_{mn}^{\ast}\\
    \end{bmatrix}
$$
</p>

我们将复共轭转置称为`Hermitian伴随「转置\共轭」`

其中如果 $ A^H = A $，则称其为`Hermitian矩阵`或者`共轭对称矩阵`

明显的，共轭转置和转置之间存在转换关系：<p> $$ A^H = ( A^\ast )^T = ( A^T )^{\ast} $$ </p>

同理的，m x n的分块矩阵的共轭转置是由A元素的共轭转置组成的 n x m 的分块矩阵：


<p>
$$
A^H =  
    \begin{bmatrix}
    A_{11}^{H}&A_{21}^{H}&\cdots&A_{m1}^{H}\\
    A_{12}^{H}&A_{22}^{H}&\cdots&A_{m2}^{H}\\
    \vdots&\vdots&&\vdots\\
    A_{1n}^{H}&A_{2n}^{H}&\cdots&A_{mn}^{H}\\
    \end{bmatrix}
$$
</p>

### 矩阵的加法，标量乘，矩阵相乘

[这个大家都知道，不如直接来看wiki上面的](https://en.wikipedia.org/wiki/Matrix_(mathematics))

明显的，常应该记住这个符合`加法交换律`和`加法结合律`

同理，`乘法结合律`、`乘法分配律`也同样要留心，特别的~~乘法交换律~~在这儿不适用

### 逆矩阵

设A是一个n阶方阵，如果我们找到一个n阶方阵，使得 $ AA^{-1} = A^{-1}A = I $ ，我们就称 $ A^{-1} $ 为其逆矩阵。

### 共轭、转置、共轭转置和逆矩阵关系

#### 1.三者均满足分配律

<p>
$$
(A+B)^{\ast} = A^{\ast} + B^{\ast} \\
(A+B)^{T} = A^{T} + B^{T} \\
(A+B)^{H} = A^{H} + B^{H} \\
$$
</p>

#### 2.三者的结合律要反向

<p>
$$
(AB)^{\ast} = B^{\ast} A^{\ast} \\
(AB)^{T} = B^{T} A^{T} \\
(AB)^{H} = B^{H} A^{H} \\
$$
</p>

#### 3.共轭、转置和共轭转置等符号均可以与求逆符号交换

<p>
$$
(A^{\ast})^{-1} = (A^{-1})^{\ast} \\
(A^{T})^{-1} = (A^{-1})^{T} \\
(A^{H})^{-1} = (A^{-1})^{H} \\
$$
</p>

我们可以用`紧凑的数学符号`来替换它： $ A^{-\ast} \quad A^{-T} \quad A^{-H} $

#### 4.Hermitian矩阵性质

对于任意矩阵A，矩阵 $ B = A^H A $ 都是Hermitian矩阵。

### 幂等矩阵「idempotent」和对合矩阵「involutory」

`幂等矩阵`：有 $ A^2 = AA = A $

`对合矩阵`：有 $ A^2 = AA = I $

### 矩阵的内积

`矩阵的内积`：

当 $ A \in C^{m \times n} $ 且 $ B \in C^{m \times p} $ 时，矩阵A和B的内积我们表示为 <p> $$ <A,B> = A^H B $$ </p>

### 矩阵的指数和对数

`矩阵指数`: <p> $$ exp(A) = \sum^{\infty}_{k=0}\frac{1}{k!} A^k $$ </p>

`矩阵对数`：<p> $$ log(I_n-A) = - \sum^{\infty}_{k=0}\frac{1}{k!} A^k $$ </p>

### 矩阵的导数和积分

`矩阵导数`: 

<p>
$$ 
\frac{dA}{dt} = \dot{A} = 
    \begin{bmatrix}
        \frac{da_{11}}{dt}&\frac{da_{12}}{dt}&\cdots&\frac{da_{1n}}{dt}\\
        \frac{da_{21}}{dt}&\frac{da_{22}}{dt}&\cdots&\frac{da_{2n}}{dt}\\
        \vdots&\vdots&&\vdots\\
        \frac{da_{m1}}{dt}&\frac{da_{m2}}{dt}&\cdots&\frac{da_{mn}}{dt}\\   
    \end{bmatrix}
$$ 
</p>

`矩阵积分`:

<p>
$$ 
\int A dt = 
    \begin{bmatrix}
        \int a_{11} dt&\int a_{12} dt&\cdots&\int a_{1n} dt\\
        \int a_{21} dt&\int a_{22} dt&\cdots&\int a_{2n} dt\\
        \vdots&\vdots&&\vdots\\
        \int a_{m1} dt&\int a_{m2} dt&\cdots&\int a_{mn} dt\\
    \end{bmatrix}
$$ 
</p>

###　矩阵函数和导数

#### 1.指数矩阵函数

<p>$$ exp(At) = I + At + \frac{A^2t^2}{2!} + \frac{A^3t^3}{3!} + \cdots $$</p>

#### 2.指数矩阵函数的导数

<p>$$ \frac{d}{dt}exp(At) = Aexp(At) = exp(At)A $$</p>

#### 3.矩阵乘积的导数

<p>$$ \frac{d}{dt}AB = \frac{dA}{dt}B + A\frac{dB}{dt} $$</p>


### 线性无关和非奇异矩阵

若一组m维向量 $ {u_1,u_2,\cdots,u_n} $是`线性无关`的，则方程：

<p>$$ c_1u_1 + c_2u_2 + \cdots + c_nu_n  = 0 $$</p>

只有零解，即：$ c_1 = c_2 = \cdots = c_n =0 $

如果找到一组不全为零的系数使得上面方程成立，即是`线性相关`的

注意：线性无关的方程组具有唯一的非零解

`奇异矩阵`：

当且仅当方阵A的n个列向量 $ a_1, a_2, \cdots , a_n $ 线性无关时，矩阵 $ A = [a_1, a_2, \cdots , a_n] $ 是`非奇异的`

## 初等行变换和阶梯形矩阵

`行等价矩阵`：仅经历初等行运算的两矩阵

`阶梯形矩阵`: 0全部出现在左下角的矩阵

`简约阶梯形`：每一非零行首项元素都是1，且每一个首1元素都是该行的唯一非0元素

### 注意：

任何一个矩阵 $ A_{m \times n} $ 都与一个且唯一一个简约阶梯形是行等价的

我们又能够引申出下面一条：

`主元位置`「pivot position」：矩阵A中与其阶梯形首项元素相对应的位置

`主元列`「pivot column」：矩阵中包含主元位置的每一列


