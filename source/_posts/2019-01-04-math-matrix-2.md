---
title: 矩阵分析与应用（二）：向量空间、内积空间和线性映射
date: 2019-01-04 20:23:01
tags: Matrix
categories: Math # 分类
thumbnail: /assets/img/posts/Math/matrix.png
---


# 向量空间、内积空间和线性映射

## 集合的基本概念

`集合`通常用花括号 $ S = \lbrace \cdot \rbrace $ 表示

几个集合运算的基本数学符号：

- $ \forall $ 表示 “对所有”
- $ x \in A $ 表示 “x是集合A中的一个元素”
- $ x \notin A $ 表示 “x不是集合A中的一个元素”
- $ \ni $ 表示“使得”
- $ \exists $ 表示“存在”

以及子集( $ \subseteq $ )、真子集( $ \subset $ )、空集( $ \varnothing $ )、并集( $ \cup $ )、交集( $ \cap $ )、和集( $ + $ )、集合差( $ - $ )和补集( $ A^\complement $ )的概念

以及最后的`笛卡尔积`的概念：

若X和Y是集合，且 $ x \in X $ 和 $ y \in Y $ ，则所有有序对「ordered pair」的集合记作：

<p>$$ X \times Y = \lbrace (x,y): x \in X , y \in Y \rbrace $$</p>

****
## 向量空间

以向量为元素的集合V称为`向量空间`，我们有下面八个公理：

- 闭合性「closure properties」
  - 加法的闭合性
  - 标量乘法的闭合性
- 加法的公理
  - 加法的交换律
  - 加法的结合律
  - 零向量的存在性
  - 负向量的存在性
- 标量乘法的公理
  - 标量乘法的结合律
  - 标量乘法的分配律
  - 标量乘法的单位率

<label class="label-theorem">定理1：</label>

如果V是一个向量空间：

- 零向量「 **0** 」是唯一的
- 对每一个向量**y**，加法的逆运算-**y**是唯一的
- 对每一个向量**y**，恒有0**y** = 0
- 对于每一个标量a,恒有a**y**=0
- 若有a**y**=0，则a=0或者**y**=0
- (-1)**y** = -**y** 

## 子空间

令V和W是两个向量子空间，如果W是V中的一个非空子集合，则则称子集合W是V的一个`子空间`

<label class="label-theorem">定理2：</label>

要让 $ R^{n} $ 的子集合W是 $ R^{n} $ 的子空间，其需要满足下列三个条件：

- 满足加法的闭合性
- 满足标量乘积的闭合性
- 满足零向量**0**是W的元素

我们考虑向量空间V的子空间A和B，若满足 $ V = A + B $ 且 $ A \cap B = \varnothing $，则称V是子空间A和B的直接求和，简称`直和`「direct sum」，记作：<p> $$ V = A \oplus B $$ </p>

<label class="label-theorem">定理3：</label>

若A和B是向量空间V的向量子空间，则 $ A + B $ 和 $ A \cap B $ 都是V的向量子空间


****

## 实内积空间「real inner product space」

`实内积空间`是满足下列条件的实向量空间E：

- 内积需要是正定(positive definite)的： $ \langle x,x \rangle > 0 ,\quad \forall x \neq 0 $
- 内积的对称性(symmetry)：$ \langle x,y \rangle = \langle y,x \rangle $
- $ \langle x,y+z \rangle = \langle x,y \rangle + \langle x,z \rangle $
- $  \langle \alpha x,y \rangle = \alpha \langle x,y \rangle$

`典范内积`(canonical inner product):

对于n阶向量空间 $ R^n $ 定义两个列向量 $ x,y $，则其典范内积为：

<p>$$ \langle x,y \rangle = \sum_{i=1}^{n}x_iy_i $$</p>

此时称 $ R^n $ 为`欧几里得`(Euclidean)空间

### 范数

`范数`「长度」：

若 $ R^n $ 是一个实内积空间，且 $ x \in R^n $ ，则 x 的范数记作 $ \lVert x \rVert $ ，并定义为：
<p>$$ \lVert x \rVert = \langle x,x \rangle ^{1/2} $$</p>

`单位向量`：长度为1的向量

向量x和y之间的距离：

<p>$$ d = \lVert x-y \rVert = \langle x-y,x-y \rangle ^{1/2} $$</p>

<label class="label-warning">特别的</label>

对于欧几里得n空间，向量范数取：

<p> $$ \lVert x \rVert _2 $$ </p>

对于范数有以下定理

<label class="label-theorem">定理4：</label>

- $ \lVert 0 \rVert = 0 $ ，且有 $ \lVert x \rVert >0 , \forall x \neq 0 $
- $ \lVert cx \rVert = \lvert c \rvert \lVert x \rVert $
- 范数符合极化恒等式(polarization identity) <p>$$ \langle x,y \rangle = \frac{1}{4} ( \lVert x+y \rVert ^2 - \lVert x-y \rVert ^2 ) \qquad \forall x,y $$</p>
- 范数满足平行四边形法则(parallelogram law) <p>$$ \lVert x+y \rVert ^2 + \lVert x-y \rVert ^2  = 2 \lVert x \rVert ^2 + 2 \lVert y \rVert ^2 \qquad \forall x,y $$</p>
- 范数服从Cauchy-Schwartz不等式：<p> $$ \lvert \langle x,y \rangle \rvert \leq \lVert x \rVert \lVert y \rVert $$ </p> 且仅在x,y线性相关时取等
- 范数满足三角不等式：<p> $$  \lVert x+y \rVert \leq \lVert x \rVert + \lVert y \rVert $$ </p>

****
## 复内积空间

`复内积空间`：

相比于实内积空间，复内积空间有些许的变化，体现在2，4条件的共轭上：

- 内积需要是严格正性的： $ \quad \forall x \neq 0  \Rightarrow \langle x,x \rangle > 0 $
- 内积的**共轭**对称性(conjugate symmetry)「Hermitian 性」：$ \langle x,y \rangle ^\ast = \langle y,x \rangle $
- $ \langle x,y+z \rangle = \langle x,y \rangle + \langle x,z \rangle $
- $  \langle c x,y \rangle = c ^\ast \langle x,y \rangle$ 对所有**复标量c**成立

相似的，有**复向量**的`典范内积`：

<p>$$ \langle x,y \rangle = x^Hy =\sum_{i}^{n} x^\ast_iy_i $$</p>

此时的n维复内积空间称为 $ C^n $ 


复内积空间中，对于范数有以下性质

<label class="label-theorem">定理5：</label>

这几个性质中只有第3个和实内积空间有些不同：

- $ \lVert 0 \rVert = 0 $ ，且有 $ \lVert x \rVert >0 , \forall x \neq 0 $
- $ \lVert cx \rVert = \lvert c \rvert \lVert x \rVert $
- 范数符合极化恒等式(polarization identity) <p>$$ \langle x,y \rangle = \frac{1}{4} ( \lVert x+y \rVert ^2 - \lVert x-y \rVert ^2 - j\lVert x+jy \rVert ^2 + j\lVert x-jy \rVert ^2) \qquad \forall x,y $$</p>
- 范数满足平行四边形法则(parallelogram law) <p>$$ \lVert x+y \rVert ^2 + \lVert x-y \rVert ^2  = 2 \lVert x \rVert ^2 + 2 \lVert y \rVert ^2 \qquad \forall x,y $$</p>
- 范数服从Cauchy-Schwartz不等式：<p> $$ \lvert \langle x,y \rangle \rvert \leq \lVert x \rVert \lVert y \rVert $$ </p> 且仅在x,y线性相关时取等
- 范数满足三角不等式：<p> $$  \lVert x+y \rVert \leq \lVert x \rVert + \lVert y \rVert $$ </p>

****
## 线性映射

`映射`：

映射本身是一个函数，如果V是欧几里得m空间 $ R^m $ 的子空间，而W是 $ R^n $ 的子空间，则 <p>$$ T:V \mapsto W $$</p> 为子空间 V到子空间W的映射「函数、变换」

对于 $ v \in V , w \in W $ ，则我们有 <p>$$ w = T(v) $$</p>

此时子空间V是映射T的`始集`(initial set)或`域`(domain)

子空间W是映射T的`终集`(final set)或`上域`(codomain)

「我们在这里将v称为原像，T(v)称为在映射下的`像`(image)或映射在v处的`值`(value)」

`值域`(range)：

一般的值域的符号为 <p> $$ T(V) = Im(T) = \lbrace T(v: v \in V) \rbrace $$ </p>

### 单射和满射

`单射`(injective)：

只要对于V中两个不同向量，他们映射后的值不同即可

`满射`(surjective):

当映射的值域等于向量空间W，即称 $ T: V \mapsto W $ 为满射

`一对一映射`(bijective)：

当一个映射同时是单射和满射时，称其为一对一映射

还有几个需要了解的映射：压缩映射、膨胀映射和矩阵变换.

<label class="label-theorem">定理6：</label>

当V和W是两个向量空间，而 $ T: V \mapsto W $ 是一`线性变换`，我们有如下性质：

- 若M是V的线性子空间，则T(M)是W的线性子空间
- 若N是W的线性子空间，则线性反变换 $ T^{-1}(N) $ 是V的线性子空间

### 同构

`同构`(isomorphic):

两个具有相同结构的向量空间E和F称为同构，使用 <p> $$ E \cong F $$ </p>

`同构映射`(isomorphism)：

两个实「或复」内积空间E和F同构，如果存在一个一对一映射 $ T: E \mapsto F $ 保持向量内积不变，即

<p>$$ \langle Tx,Ty \rangle = \langle x,y \rangle \qquad  \forall x,y \in E $$</p>

则这样的映射T为向量空间的同构映射









