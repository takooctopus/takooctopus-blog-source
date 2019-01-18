---
title: 矩阵分析与应用（七）：逆矩阵
date: 2019-01-17 22:08:35
tags: Matrix
categories: Math # 分类
thumbnail: /assets/img/posts/Math/matrix.jpg
---

# 逆矩阵

对于一个 $ n \times n $ 的矩阵，只有当它同时具有n个线性无关的列向量和n个线性无关的行向量是，能称其为`非奇异`的。此时它只对零输入产生零输出。

而一个非奇异矩阵，一定存在其`逆矩阵`，即当 $ AB = BA = I $ 时，称矩阵 $ B $ 是矩阵 $ A $ 的逆矩阵，记作 $ A^{-1} $

`伴随矩阵(adjoint matrix)`:

将正方矩阵 $ A $ 的所有元素 $ a_{ij} $ 由它的`余子式` $ A_{ij} $ 代替，并`转置`，所得到的矩阵称为 $ A $ 的伴随矩阵：

<p>
$$
    adj(A) = 
    \begin{bmatrix}
    A_{11}&A_{21}&\cdots&A_{n1}\\
    A_{12}&A_{22}&\cdots&A_{n2}\\
    \vdots&\vdots&&\vdots\\
    A_{1n}&A_{2n}&\cdots&A_{nn}\\
    \end{bmatrix}
$$
</p>

若行列式 $ det(A) \neq 0 $ ，即存在唯一一个逆矩阵 $ A^{-1} $ ：

<p>
$$
    A^{-1}  = \frac{adj(A)}{det(A)} = 
    \frac{1}{\lvert A \rvert}
    \begin{bmatrix}
    A_{11}&A_{21}&\cdots&A_{n1}\\
    A_{12}&A_{22}&\cdots&A_{n2}\\
    \vdots&\vdots&&\vdots\\
    A_{1n}&A_{2n}&\cdots&A_{nn}\\
    \end{bmatrix}
$$
</p>

## 逆矩阵的性质

- $ [adj(A)]^T = adj(A^T) $
- $ A^{-1}A = AA^{-1} = I $
- $ A^{-1} $ 唯一
- $ \lvert A^{-1} \rvert = \frac{1}{\lvert A \rvert} $
- 逆矩阵非奇异
- $ (A^{-1}) ^{-1} = A $
- 复共轭转置可写为 $ A^{-H} $
- 如果 $ A^H = A $ ，则 $ (A^{-1}) ^H = A^{-1} $ 
- $ (A^{\ast}) ^{-1} = (A^{-1}) ^{\ast} $
- $ (AB)^{-1} = B^{-1} A^{-1} $
- 对于对角矩阵 $ A = diag(a_1,a_2,\cdots,a_m) $ ，有逆矩阵:

<p>$$ A^{-1} = diag(a_1^{-1},a_2^{-1},\cdots,a_m^{-1}) $$</p>

- 若 $ A $ 为正交矩阵，有 $ A^{-1} = A^T $
- 若 $ A $ 为[酉矩阵](/2019-01-08-math-matrix-4/#酉不变)，有 $ A^{-1} = A^H $

****
## 矩阵的求逆引理

`Sherman-Morrison公式`：

令 $ A $ 是一个 $ n \times n $ 的可逆矩阵，并且 $ x $ 和 $ y $ 是两个 $ n \times 1 $ 向量，使得 $ (A+xy^H) $ 可逆，则有：

<p>$$ (A+xy^H)^{-1} = A^{-1} - \frac{A^{-1}xy^H A^{-1}}{1+y^H A^{-1} x} $$</p>

我们可以对其做一个简单的证明：

<label class="label-theorem">证明：</label>

由于：

<p>$$ (A+xy^H) = A(I+A^{-1}xy^H) $$</p>

对其同时求逆，有：

<p>$$ (A+xy^H)^{-1} = (I+A^{-1}xy^H) ^{-1} A^{-1} $$</p>

而我们知道 $ (I+B) $ 如果可逆，且 $ I \neq B $ 的话，我们可以表示为：

<p>$$ (I+B)^{-1} = I - B + B^2 - B^3 + \cdots $$</p>

带入原式，即有：

<p>$$ (I+A^{-1}xy^H) ^{-1} = I - A^{-1}xy^H + (A^{-1}xy^H)^2 -(A^{-1}xy^H)^3 + \cdots $$</p>

明显的，这是一个等比数列的求和问题，带回得到：

<p>
$$
\begin{split}
(A+xy^H)^{-1}   &= A^{-1} - A^{-1}xy^HA^{-1} + A^{-1}x(y^HA^{-1}x)y^HA^{-1} - \cdots \\
                &= A^{-1} - A^{-1}x (\sum^{i=0}_{\infty}y^HA^{-1}x) y^HA^{-1} \\
                &= A^{-1} - \frac{A^{-1}xy^HA^{-1}}{1+y^HA^{-1}x}
\end{split}
$$
</p>

<label class="label-warning">我们还有更为一般的式子：</label>

`Sherman–Morrison–Woodbury 公式`

对于 $ A \in K^{n \times n} $ 是域 $ K $ 上的非奇异矩阵，且 $ U,V \in K^{n \times k} $ ，且已知 $ A+UV^H $ 非奇异，则我们有`Sherman–Morrison–Woodbury 公式`：

<p>$$ (A+UV^H)^{-1} = A^{-1} - A^{-1}U(I_k + V^HA^{-1}U)^{-1}V^HA^{-1} $$</p>

### 矩阵之和的求逆公式(Woodbury公式)：

`Woodbury公式`：

我们根据上面的矩阵求逆引理可以推广为矩阵之和的求逆公式：

<p>
$$
\begin{split}
(A+UBV)^{-1}    &= A^{-1} - A^{-1}UB(B+BVA^{-1}UB)^{-1}BVA^{-1} \\
                &= A^{-1} - A^{-1}U(I+BVA^{-1}U)^{-1}BVA^{-1} \\
                &= A^{-1} - A^{-1}U(B^{-1}+VA^{-1}U)^{-1}VA^{-1}
\end{split}
$$
</p>

换成加法也是相似的：

<p>$$ (A-UV)^{-1} = A^{-1} + A^{-1}U(I-VA^{-1}U)^{-1}VA^{-1} $$</p>

这其中的 $ I-VA^{-1}U $ 被称作`容量矩阵(capacitance matrix)` 

<label class="label-warning">矩阵之和的逆矩阵还有其他形式：</label>

其中的车别就是 $ UBV $ 中间取的断点不同而已


****
## 分块矩阵的求逆

其实这是延续于上面的矩阵和求逆公式的

### 当矩阵 $ A,D $ 均可逆时

<p>
$$
\begin{bmatrix}
    A & U \\
    V & D \\
\end{bmatrix}^{-1}
=
\begin{bmatrix}
    (A-UD^{-1}V)^{-1} & -(V-DU^{-1}A)^{-1} \\
    (U-AV^{-1}D)^{-1} & (D-VA^{-1}U)^{-1} \\
\end{bmatrix}
$$
</p>

我们可以通过 $ A,U,V,D $ 的维数去记忆这个公式。

此外，如果 $ D $ 不可逆，我们应该用通用的方式去表示 $  (A-UD^{-1}V) ^{-1} $ ，即：

<p>$$  (A-UD^{-1}V)^{-1} =  A^{-1} - A^{-1}U(D+VA^{-1}U)^{-1}VA^{-1} $$</p>

<label class="label-warning">注意次对角线 $ U,V $：</label>

次对角线上的并非方阵，并不能带入`Sherman–Morrison公式`，但可以提取公因式。

而分块矩阵的求逆操作在推导斜投影矩阵的过程中有重要作用。

****
## Woodbury公式的典型应用

假设 $ J_n $ 是一个n维全1方阵，且 $ V $ 为：

<p>
$$
V = 
\begin{bmatrix}
    a & b & \cdots & b \\
    b & a & \cdots & b \\
    \vdots & \vdots & \ddots & \cdots \\
    b & b & \cdots & a \\
\end{bmatrix}
= [(a-b)I_n + bJ_n] = (a-b)(I_n+\frac{b}{a-b}J_n)
$$
</p>

这里我们用`Woodbury公式`，能够得到逆矩阵：

<p>$$ V^{-1} = \frac{1}{a-b}(I_n +\frac{b}{a-b}J_n) ^{-1} = \frac{1}{a-b}[I_n - \frac{b}{a+(n-1)b}J_n] $$</p>

### 解方程式过程

假设 $ A,U,V $ 均为n阶方阵，我们要求解 $ (A-UV)x = b $ 的解：

- 求解矩阵方程 $ Ay=b $ 得到 $ y $
- 求解矩阵方程 $ Aw_i = u_i $ ，得到 $ w_i $ ，构造矩阵 $ W = [w_1, w_2, \cdots, w_n] $ ，即此时 $ W = A^{-1}U $
- 构造矩阵 $ C = I - VW $ 和向量 $ Vy $ ，求解线性方程 $ Cz =Vy $ ，得到 $ z $
- 矩阵方程 $ (A-UV)x = b $ 的解就是 $ x = y + Wz $


****
## Hermitian矩阵分块求逆引理

`Hermitian矩阵分块求逆引理`：

我们知道Hermitian矩阵的性质，即我们可以将其用分块矩阵的形式表示出来：

<p>
$$
R_{m+1} =  
\begin{bmatrix}
    R_m & r_m \\
    r_m^H & \rho_m
\end{bmatrix}
$$
</p>

我们可以从 $ R_m^{-1} $ 去推出 $ R_{m+1}^{-1} $

先定义 $ Q_{m+1} $ ，有：

<p>
$$
Q_{m+1} =  
\begin{bmatrix}
    Q_m & q_m \\
    q_m^H & \alpha_m
\end{bmatrix}
$$
</p>

且 $ R_{m+1} $ 和 $ Q_{m+1} $ 有以下关系：

<p>
$$
R_{m+1}Q_{m+1} =  
\begin{bmatrix}
    R_m & r_m \\
    r_m^H & \rho_m
\end{bmatrix}
\begin{bmatrix}
    Q_m & q_m \\
    q_m^H & \alpha_m
\end{bmatrix}
=
\begin{bmatrix}
    I_m & 0_m \\
    0_m^H & 1
\end{bmatrix}
$$
</p>

这样会出4个方程：

<p>
$$
\begin{split}
R_mQ_m + r_mq_m^H = I_m \\
r_m^HQ_m + \rho_mq_m^H = 0_m^H \\
R_mq_m + r_m\alpha_m = 0_m \\ 
r_m^Hq_m + \rho_m\alpha_m = 1 \\
\end{split}
$$
</p>

从第三个方程可以得到 $ q_m $ 的表示：

<p>$$ q_m = -\alpha_m R_m^{-1} r_m $$</p>

再带入第4个方程得到：

<p>$$ \alpha_m = \frac{1}{\rho_m - r_m^{H}R_{m}^{-1}r_m } $$</p>

即我们可以知道 $ q_m $ 的真实表达：

<p>$$ q_m = \frac{-R_{m}^{-1}r_m}{\rho_m - r_m^{H}R_{m}^{-1}r_m } $$</p>

同理带回得到 $ Q_m $ :

<p>$$ Q_m = R_m^{-1} + \frac{R_{m}^{-1}r_m (R_{m}^{-1}r_m)^H}{\rho_m - r_m^{H}R_{m}^{-1}r_m } $$</p>

<label class="label-theorem">简化操作</label>

为了简化，我们需要定义两个新的符号：

<p>$$ b_m \overset{def}{=} [b_0^{(m)},b_1^{(m)},\cdots,b_{m-1}^{(m)}]^T = - R_{m}^{-1}r_m $$</p>

<p>$$ \beta_m = \rho_m - r_m^HR_m^{-1}r_m = \rho_m + r_m^Hb_m $$</p>

此后我们就将其简化为了：

<p>
$$
\alpha_m = \frac{1}{\beta_m} \\
q_m = \frac{1}{\beta_m}b_m \\
Q_m = R_m^{-1} + \frac{1}{\beta_m}b_mb_m^H \\
$$
</p>

即我们最终得到了 $ R_{m+1} $ 的逆矩阵：

<p>
$$
R_{m+1}^{-1} = Q_{m+1}
\begin{bmatrix}
    R_m^{-1} & 0_m \\
    0_m^T & 0 \\
\end{bmatrix}
+ \frac{1}{\beta_m}
\begin{bmatrix}
    b_mb_m^H & b_m \\
    b_m^H & 1
\end{bmatrix}
$$
</p>

这就是从 $ R_m $ 求 $ R_{m+1} $ 的秩1修正公式，即`Hermitian矩阵分块求逆引理`。