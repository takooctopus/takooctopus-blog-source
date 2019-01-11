---
title: 矩阵分析与应用（四）：内积和范数
date: 2019-01-08 15:26:24
tags: Matrix
categories: Math # 分类
thumbnail: /assets/img/posts/Math/matrix.jpg
---


# 内积和范数

##  向量的内积和范数

我们令 $ V $ 是复向量空间

`内积公理`：

函数 $ V \times V \mapsto C $ 称为内积

- $ \langle x , x \rangle \geq 0 $ (非负性)
- $ \langle x , x \rangle = 0 $ 仅当 $ x = 0 $ (正性)
- $ \langle x + y , z \rangle = \langle x , z \rangle + \langle y , z \rangle $ (可加性)
- $ \langle cx , y \rangle = c^{\ast} \langle x , y \rangle $ (齐次性)
- $ \langle x , y \rangle = \langle y , x \rangle ^{\ast} $ (Hermitian性)

`范数公理`：

同理的，范数 $ \lVert X \rVert : V \mapsto R $ 称为向量x的范数

- $ \lVert x \rVert \geq 0 $ (非负性)
- $ \lVert x \rVert = 0 $ 仅当 $ x = 0 $ (正性)
- $ \lVert cx \rVert = \lvert c \rvert \lVert x \rVert $ (齐次性)
- $ \lVert x + y \rVert \leq \lVert X \rVert + \lVert y \rVert $ (三角不等式)

### 常数向量的内积和范数

`内积的定义`：

<p>$$ \langle x,y \rangle = x^Hy = \sum^{m}_{i=1}x_i^{\ast}y_i $$</p>

我们就很容易用上面的来定义两个向量之间的`夹角`:

<p>$$ cos \theta \overset{def}{=} \frac{\langle x,y \rangle}{\langle x,x \rangle \langle y,y \rangle}  = \frac{x^Hy}{\lVert x \rVert \lVert y \rVert } $$</p>

- $ l_1 $ 范数
    <p> $$ \lVert x \rVert _1 \overset{def}{=} \lvert \sum^{m}_{i=1} x_i \rvert = \lvert x_1 \rvert + \lvert x_2 \rvert + \cdots + \lvert x_m \rvert $$ </p>
- $ l_2 $ 范数
    <p> $$ \lVert x \rVert _2 \overset{def}{=} ( \lvert x_1 \rvert ^2 + \lvert x_2 \rvert ^2 + \cdots + \lvert x_m \rvert ^2 )^{1/2} $$ </p>
- $ l_{\infty} $ (无穷)范数
    <p>$$ \lVert x \rVert _{\infty} \overset{def}{=} max( \lvert x_1 \rvert , \lvert x_2 \rvert , \cdots , \lvert x_m \rvert ) $$</p>
- $ l_p (H\ddot{o}lder) $ 范数
    <p>$$ \lVert x \rVert _{p} \overset{def}{=} (\sum^{m}_{i=1} \lvert x_i \rvert ^p) ^{1/p} \qquad p \geq 1 $$</p>

明显的，无穷范数是 $H\ddot{o}lder$范数的极限形式


### 酉不变

`酉不变`：

范数 $ \lVert x \rVert $ 是如满足：$ \forall x \in C^m , U \in C^{m \times m} $ 有 $ \lVert Ux \rVert = \lVert x \rVert $  则称其为酉不变。此时 $ U $ 称为`酉矩阵`

### 函数向量的内积和范数

与常数向量相似，函数向量的内积我们可以定义为一个累次积分：

`内积`：

<p>$$ \langle x(t),y(t) \rangle \overset{def}{=} \int^{b}_{a} x^H(t)y(t) dt $$</p>

`夹角`;

<p>$$ cos \theta \overset{def}{=} \frac{\int^{b}_{a} x^H(t)y(t) dt}{\lVert x(t) \rVert \lVert y(t) \rVert } $$</p>

`范数`:

<p>$$ \lVert x(t) \rVert = ( \int^{b}_{a} x^H(t)x(t) dt )^{1/2} $$</p>

### 随机向量的内积和范数

与上面相似，但这次 $ x(\xi) , y(\xi) $ 是样本变量 $ \xi $ 的随机向量

`内积`：

<p>$$ \langle x(\xi),y(\xi) \rangle \overset{def}{=} E \lbrace x^H(\xi)y(\xi) \rbrace $$</p>

`范数`：

<p>$$ \lVert x(\xi) \rVert _{2} \overset{def}{=} E \lbrace x^H(\xi)x(\xi) \rbrace $$</p>

<label class = 'label-warning'> 关于正交： </label>

两个$ m \times 1, n \times 1 $维列随机向量要正交，需要他们的任意元素相互正交，即其互相关矩阵为零矩阵：

<p>$$ E \lbrace x(\xi)y^H(\xi) \rbrace = O_{m \times n}$$</p>


`Pythagorean`定理：

若 $ x \perp y $，则有 $ \lVert x+y \rVert ^2 =  \lVert x \rVert ^2 + \lVert y \rVert ^2 $ 成立

`Cauchy-Schwarts`(柯西-施瓦茨)不等式：

<p> $$ \lVert \langle x,y \rangle \rVert \leq \lVert x \rVert \lVert y \rVert  $$ </p>

`平行四边形`法则：

<p>$$ \lVert x+y \rVert ^2 + \lVert x-y \rVert ^2 = 2 \lVert x \rVert ^2 + 2 \lVert y \rVert ^2$$</p>

****
## 向量的相似度

`类型定义`：

M个类型的模式，记作 $ \omega_1 , \omega_2 , \cdots , \omega_M $

通过已知的类型属性的观测样本，抽出M个样本模式向量： $ s_1, s_2 ,\cdots, s_M $

未知模式向量 $ x $

`更相似`：

如果我们定义 $ (x,s_i) $ 是模式向量x与 $ s_i $ 之间相似关系的符号(notation)，当有

<p>$$ (x,s_i) \leq (x,s_j) $$</p>

我们称未知模式向量 $ x $ 与样本模式向量 $ s_i $ 更相似

为此我们定义了`相似度`(similarity)和`相异度`(dissimilarity)

### 相似度和相异度

`Euclidean`(欧几里德)距离
  
明显的，欧式距离就是其直线距离，也是欧几里德范数，我们记作 $ \sqrt{(x-s_i)^T(x-s_i)} $

`Mahalanobis`(马哈拉诺比斯)距离

马氏距离更加的表现了数据的协方差距离，其考虑了各种特性之间的联系，且其实尺度无关(scale-invariant)的

我们很容易就能得到其中里面几个参数：

均值向量： $ m = \frac{1}{N} \sum^{N}_{k=1}s_i $  

协方差矩阵： $ C = \frac{1}{N} \sum^{N}_{k=1} (s_i-m)(s_i-m)^T $

马氏距离： $ D(m,x) = (x-m)^TC(x-m) $

这样我们只需要找到和 $ D(m,x) $ 值最相近的 $ D(m,s_i) $ 即可

`角度的使用`：

余弦值的使用: $ S(s_i,x) = cos(\theta_i) = \frac{x^Ts_i}{\lVert x \rVert _2 \lVert s_i \rVert _2} $

`Tanimoto`测量：

Tanimoto测量系数： $ S(s_i,x) = \frac{x^T s_i}{x^T x+s_i^T s_i + x^T s_i} $

<label class="label-theorem">雅卡尔指数</label>

这个是另一个`Jaccard index`(雅卡尔指数)，即交并比的引申：

原本的定义是： $ J(A,B) = \frac{\lvert A \cap B \rvert}{\lvert A \cup B \rvert} = \frac{\lvert A \cap B \rvert}{\lvert A \rvert + \lvert B \rvert - \lvert A \cap B \rvert} $

****
## 向量范数与Lyapunov函数

`Lyapunov`稳定性原理：

若对于连续系统 $ \dot{x} = f(x) $ 或者离散系统 $ x_{k+1} = f(x_R) $ 存在一个函数 $ V(x) $ 具有平衡点 $ x = 0 $ ，且 $ V $ 在整个 $ R^n $
内满足条件：

- $ V $ 是正定和径向无界函数

- 对于 $ x \neq 0 $

<p>$$ DV =  \lim_{\triangle t \to 0} \sup \frac{V(x(t+ \triangle t))- V(x(t))}{\triangle t} < 0 \qquad \text{连续系统} $$</p> 

<p>$$ \triangle V = V(x_{k+1}) - V(x_k) < 0 \qquad 离散系统 $$</p>

则平衡点 $ x = 0 $ 是全局渐进稳定的

在n维向量空间，我们考虑使用向量范数 $ V(x) = \lVert W_x \rVert $ 

其中：$ W = [\omega_1, \omega_2, \cdots , \omega_n ] $ 是 $ m \times n $ 矩阵，且 $ m > n $ 及 $ rank(W) = n $

### $ l_p $ 范数作用

$ l_p $ 范数在其中构成了一类特殊的向量范数：

`Euclidean`范数：

<p>$$ V(x) = \lVert Wx \rVert _2 = ( \sum_{i} \lvert w_i^Tx \rvert ^2 )^{1/2} $$</p>

`无穷`范数：

<p>$$ V(x) = \lVert Wx \rVert _{\infty} = \lim_{p \to \infty} (\sum_{i} \lvert w_i^Tx \rvert ^p) ^{1/p} = \max_{i} \lbrace w_i^Tx \rbrace  $$</p>

<label class="label-theorem"> 充要条件 </label>

`Lyapunov函数`：

函数 $ V(x) = \lVert Wx \rVert $ 是系统 $ \dot{x} = Ax $ 的`Lyapunov函数`，当且仅当矩阵 $ W $ 是下式的解：

<p>$$ WA-QW = O $$</p>

且 $ \mu (Q) < 0 $

`对数矩阵范数` $ \mu (Q) $ :

<p>$$ \mu (Q) = \lim_{\triangle t \to 0^+} \frac{\lVert I + \triangle t Q \rVert - 1}{\triangle t} $$</p>

<label class="label-warning"> 注意： </label>

对数矩阵范数可以是负数，这一点可能与实际的矩阵范数非负性相`违背`

`平方性质`：

如果函数是Lyapunov函数，则它的平方：

<p>$$ V^2(x) = \lVert Wx \rVert _2 ^2 = \sum_{i=1}^{n} (\omega_i^T x)^2 = x^T W^T Wx $$</p>

我们可以看见这函数是一个二次型 $ x^T W^T Wx $

我们令 $ R = W^T W $ ，我们要使 $ V^2(x) $ 这个式子是 $ \dot{x} = Ax $ 的Lyapunov函数，需要满足：

<p>$$ A^TR+RA = -\tilde{Q} $$</p>

这个方程的解 $ \tilde{Q} $ 是一个正定对称矩阵

<label class="label-theorem"> 等价定理 </label>

我们知道下面两个集合`等价`:

<p>$$ L_1 = \lbrace R \in R^{n \times n} | A^TR + RA = - \tilde{Q} ,其中 \tilde{Q},R > 0, \tilde{Q} 对称 \rbrace $$</p>

<p>$$ L_2 = \lbrace R \in R^{n \times n} | R = W^TW, WA-QW = O ,其中 \mu _2 (Q) < 0, rank(W) = n \rbrace $$</p>

****
## 矩阵的范数与内积

矩阵的范数仍旧符合开始的几个性质，即`非负性`，`正定性`，`三角不等式`和`柯西-施瓦茨不等式`。

### 典型的矩阵范数

`Frobenius`范数：

<p>$$ \lVert A \rVert _F \overset{def}{=} (\sum^{m}_{i=1} \sum^{n}_{j=1} \lvert a_{ij} \rvert)^{1/2} $$</p>

相当于将整个矩阵的元素排到一行中

`Minkowski` $ l_p $ 范数：

<p>$$ \lVert A \rVert _p \overset{def}{=}  \max_{x \neq 0} \frac{\lVert Ax \rVert _p}{\lVert x \rVert _p} $$</p>

`行和`(row-sum)范数：

<p>$$ \lVert A \rVert_{row} = \max_{1 \leq i \leq m} \lbrace \sum^{n}_{j=1} \lvert a_{ij} \rvert \rbrace $$</p>

`列和`(colum-sum)范数：

<p>$$ \lVert A \rVert_{col} = \max_{1 \leq j \leq n} \lbrace \sum^{m}_{i=1} \lvert a_{ij} \rvert \rbrace $$</p>

`谱`(spectrum)范数「最大奇异值、算子」范数：

<p>$$ \lVert A \rVert_{spectrum} = \sigma_{max} = \sqrt{\lambda_{max}} $$</p>

这其中，$ \sigma_{max} $ 称为矩阵 $ A $ 的最大奇异值

`Mahalanobis`范数：

<p>$$ \lVert A \rVert_{\Omega} = \sqrt{tr(A^H \Omega A)} $$</p>

且 $ \Omega $ 是正定矩阵