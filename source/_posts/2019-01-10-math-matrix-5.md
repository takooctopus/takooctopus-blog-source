---
title: 矩阵分析与应用（五）：基与Gram-Schmidt正交化
date: 2019-01-10 23:13:14
tags: Matrix
categories: Math # 分类
thumbnail: /assets/img/posts/Math/matrix.jpg
---

# 基与Gram-Schmidt正交化

## 向量子空间的基

我们知道，$ R^n $ 空间的多个向量的所有线性组合也属于 $ R_n $

假设 $ R_n $ 中的几个 $ n \times 1 $ 的向量，假设其是线性无关的，即最终可以通过他们去`张成(span)`或`生成(generate)`Euclidean $ n $ 空间 $  R^n $ 的一个子空间

`张成集(spanning set)`或`生成元(generator)`：

向量 $ x_1, x_2, \cdots, x_d $ 的所有线性组合的集合称为由 $ x_1, x_2, \cdots, x_d $ 张成（或生成）的子空间或闭包(closure)，我们记作：

<p>$$ W = Span \lbrace x_1,x_2,\cdots,x_d \rbrace = Close \lbrace x_1,x_2,\cdots,x_d \rbrace $$</p>

<label class = "label-warning"> 注意： </label>

一个子空间的张成集并不是唯一的

子空间所有向量的集合称为`平凡张成集`或`平凡生成集`

我们需要找到子空间 $ W $ 的生成元的最小集合，即只需要d个向量可生成。

`子空间的基向量`：

生成子空间 $ W $ 的线性无关向量 $ \lbrace \mu_1, \mu_2, \cdots, \mu_d \rbrace $ 称为子空间W 的基向量(base vector)

生成子空间 $ W $ 的基向量的个数称为子空间 $ W $ 的`维数`

<p>$$ d = dim(Span \lbrace \mu_1, \mu_2, \cdots, \mu_d \rbrace) $$</p>

同理： $ \lbrace \mu_1, \mu_2, \cdots, \mu_d \rbrace $ 也仅是子空间的一组基，而非唯一基

`对偶基`:

如果 $ \lbrace \alpha_1, \alpha_2, \cdots, \alpha_n \rbrace $ 和 $ \lbrace \beta_1, \beta_2, \cdots, \beta_n \rbrace $ 是两组不同的基，且有 $ \alpha^H_i \beta_i = 0 $ ，则称其中一组基石另一组基的对偶基

`正交基向量`：

令 $ \lbrace x_1, x_2, \cdots, x_n \rbrace $ 是子空间 $ Span \lbrace x_1, x_2, \cdots, x_n \rbrace $ 的基向量，且满足正交条件：

<p>$$ \langle x_i,x_j \rangle = x^T_i x_j = 0 \qquad \forall i \neq j $$</p>

则称这些基向量为正交基向量

`标准正交基(orthonomal basis vectors)`：

若正交基向量的范数等于1

<label class = "label-example"> 傅里叶级数的正交基 </label>

如果标量函数 $ x(t) $ 是周期为T的函数，且是绝对平方可积分的，即 $ \int^{\infty}_{-\infty} \lvert x(t) \rvert ^2 dt < \infty $

即我们可以将其展开为傅里叶级数：

<p>$$ x(t) = \sum^{\infty}_{-\infty} c_n e^{jn\omega_0 t}  \qquad  其中 \omega_0 = \frac{2\pi}{t} $$</p>

即期正交基为：

<p> $$ 1, e^{j\omega_0 t}, e^{j2\omega_0 t} ,e^{j3\omega_0 t}, \cdots $$ </p>

<label class = "label-example"> 离散小波的正交基 </label>

对于单个原像小波函数 $ h(\cdot) $， 我们对其进行伸缩和平移参数，有基函数：

<p>$$ h_{a,b}(t) = \frac{1}{\sqrt{a}} h(\frac{t-b}{a}) ,\qquad a \in R^{+} $$</p>

这个能够用以构造时间很短的高频基函数或时间长的低频基函数

我们也由此能够定义信号 $ x(t) $ 的离散小波变换：

<p>$$ WT_x{m,n} = a_0^{-m/2}\int^{\infty}_{-\infty} x(t)h(a_0^{-m}t-nb_0)dt $$</p>

标准正交基由Haar基：

<p>
    $$
    h(t) = 
    \left\{
    \begin{array}{l}
    1 & 0 \leq t \le 1/2 \\
    -1 & 1/2 \leq t \le 1 \\
    0 & 其它 \\
    \end{array}
    \right.
    $$
</p>

并取 $ a_0 = 2, b_0 = 1 $ 得到

`向量子空间的性质`

- 如果 $ W_1 $ 和 $ W_2 $ 都是向量空间 $ V $ 中的两个子空间，则它们的交集 $ W_1 \cap W_2 $ 也是 $ V $ 的子空间
- 如果 $ W_1 $ 和 $ W_2 $ 都是向量空间 $ V $ 中的两个子空间，则它们的交集 $ W_1 +    W_2 $ 也是 $ V $ 的子空间


****
## Gram-Schmidt 正交化

`Gram-Schmidt 正交化`【上课都讲过的】：

令 $ \lbrace x_1, x_2, \cdots, x_n \rbrace $ 是p维向量子空间W的任意一组基，即子空间W的标准正交基 $ \lbrace u_1, u_2, \cdots, u_n \rbrace $ 可以通过Gram-Schmidt 正交化变换得到：

<p>
$$ 
p_1 = x_1, \qquad u_1 = \frac{p_1}{\lVert p_1 \rVert} = \frac{x_1}{\lVert x_1 \rVert} 
p_k = x_k - \sum^{k-1}_{i=1}(u_i^H x_k)u_i , \qquad u_k = \frac{p_k}{\lVert p_k \rVert}
$$
</p>

我们将这个称为经典的Gram-Schmidt 正交化算法，其数值性能可能不太好

<label class="label-theorem"> $ Bj\ddot{o}rck $ 的修正Gram-Schmidt正交化</label>

一步步看，第一步是相同的

<p>$$ u_1 = \frac{x_1}{\lVert x_1 \rVert}  $$</p>

然后我们就对剩余的其余数据向量 $ \lbrace x_2, x_3, \cdots, x_n \rbrace $ 进行修正：

<p>$$ x_i^{(1)} = x_i - (u_1^H x_i)u_1 ,  \qquad i= 2,3,\cdots,n $$</p>

上面的均与 $ u_1 $ 正交

然后对第二个已修正的向量 $ x_2^{(1)} $ 单位化：

<p>$$ u_2 = \frac{x_2^{(1)}}{\lVert x_2^{(1)} \rVert}  $$</p>

再对后面的向量进行第二次修正

最终第k个正交化向量定义成：

<p>$$ u_k = \frac{x_k^{(k-1)}}{\lVert x_k^{(k-1)} \rVert}  $$</p>

并得到被修正的数据向量：

<p>$$  x_i^{(k)} = x_i^{(k-1)} - (u_k^H x_i^{(k-1)})u_k ,  \qquad i= k+1,k+2,\cdots,n $$</p>

直到最后将所有向量正交化结束为止

