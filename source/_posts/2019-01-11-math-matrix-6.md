---
title: 矩阵分析与应用（六）：矩阵的标量函数
date: 2019-01-11 14:59:46
tags: Matrix
categories: Math # 分类
thumbnail: /assets/img/posts/Math/matrix.jpg
---

# 矩阵的标量函数


## 二次型

`二次型`：

对于任何正方矩阵 $ A $ ，其有二次型 $ x^H A x $ 是一个实标量，算其为x的二次型函数

做出推广，对于 $ x = [x_1, x_2, \cdots, x_n]^T $ ，且 $ x \times n $ 矩阵A的元素为 $ a_{ij} $ ，则二次型为：

<p>
$$
\begin{split}
    x^T A x  &= \sum^{n}_{i=1}\sum^{n}_{j=1} x_i x_j a_{ij}\\
                            &= \sum^{n}_{i=1}a_{ii}x_i^2 + \sum^{n-1}_{i=1}\sum^{n}_{j=i+1} (a_{ij} + a_{ji})x_i x_j \\
\end{split}
$$
</p>

<label class="label-warning"> 警告：二次型的多样性</label>

对于一个二次型函数，可能存在多个矩阵 $ A $ ，使得他们拥有相同的二次型

但是其中仅有唯一的一个`对称矩阵` 

<label class="label-theorem"> 我们在此假定A是实对称的或复对称的(Hermitian)</label>

我们在此还可以根据其二次型的正负定义其正定性：

- 正定矩阵
- 半正定矩阵
- 负定矩阵
- 半负定矩阵
- 不定矩阵

<label class="label-warning"> 警告：正定矩阵和正矩阵</label>

`正定矩阵`由其二次型的正负决定

`正矩阵`由其所有元素的正负决定


### Debreu 引理

`Debreu 引理`：

给定 $ m \times n $ 矩阵 $ J $ 和  $ n \times n $ 对称矩阵 $ H $

且二次型 $ x^H H x > 0 $ 对所有满足 $ Jx = 0 $ 的向量 $ x $ 成立

`Thus`：当且仅当存在一个有限大的数 $ \bar{\rho} \geq 0 $ ，使得 $ H + \rho J^T J $ 对所有的 $ \rho > \bar{\rho} $ 都是正定的


****
## 矩阵的迹

`矩阵的迹`：

$ n \times n $ 矩阵A对角线元素之和称为A的迹(trace)，记作 $ tr(A) $ ，即：

<p>$$ tr(A) = \sum^{n}_{i=1} a_{ii} $$</p>

### 迹的性质

`迹的性质`：

- 可脱性： $ tr(A \pm B) = tr(A) \pm tr(B) $
- 标量乘法 ： $ tr(cA) = ctr(A) $
- 标量乘法和可脱性结合
- 转置、复数共轭和复共轭转置：

<p>
$$
\begin{split}
    tr(A^T) &= tr(A) \\
    tr(A^{\ast}) &= [tr(A)]^{\ast} \\
    tr(A^H) &= [tr(A)]^{\ast}
\end{split}
$$
</p>

- 迹是相似不变量： $ tr(A_{m \times n}B_{n \times m}) = tr(B_{n \times m}A_{m \times n}) $
- 矩阵的相似等性质，若AB均为 $ m \times m $ 矩阵，且B非奇异的：

<p>$$ tr(B A B^{-1}) = tr(B^{-1} A B) = tr(A) $$</p>

- 零矩阵性质： $ tr(A^H_{m \times n} A = 0 \leftrightarrow A = O_{m \times n}) $
- $ X^H A x = tr(Axx^H) ,\qquad y^H x = tr(xy^H) $
- 分块矩阵的迹：

<p>
    $$
    tr
    \begin{bmatrix}
        A & B \\
        C & D \\
    \end{bmatrix}
    = tr(A) + tr(D)
    $$
</p>

- 复共轭转置的相乘 $ tr(A^H A) = tr(A A^H) = \sum_{i=1}^{n} \sum_{j=1}^{n} a_{ij}^{\ast} a_{ji} $ 
- 迹等于特征值之和：

<p>$$ tr(A) = \lambda_{1} + \lambda_{2} + \cdots + \lambda_{n} $$</p>

- k次矩：

<p>$$ tr(A^k) = \sum^{n}_{i=1} \lambda^k_i $$</p>

### 迹的不等式

`迹的不等式`：

- 明显的，由于复转置矩阵性质有： $ tr(A^H A) = tr(A A^H) \geq 0 $
- 若对A,B两 $ m \times n $ 阶矩阵：

<p>
$$
\begin{split}
    tr[(A^T B)^2] &\leq tr(A^T A) tr(B^T B) \qquad (Cauchy-Schwartz不等式)\\
    tr[(A^T B)^2] &\leq tr(A^T A B^T B) \\
    tr[(A^T B)^2] &\leq tr(A A^T B B^T) \\
\end{split}
$$
</p>

- Schur不等式： $ tr(A^2) \leq tr(A^T A) $
- $ tr[ (A+B) (A+B)^T ] \leq 2[tr(A A^T) + tr(B B^T)] $
- 若A和B为 $ m \times m $ 对称矩阵，则有：$ tr(AB) \leq \frac{1}{2}tr(A^2 + B^2) $

我们由此可以使用矩阵的迹去定义 $ m \times m $ 矩阵的的[Frobenius范数](/2019-01-08-math-matrix-4/#典型的矩阵范数)：

<p>$$ \lVert x \rVert _F = \sqrt{tr(A^T A)} = \sqrt{tr(A A^T)} $$</p>


****
## 行列式

`行列式`：

大家也很熟悉了，行列式定义：

<p>
$$
    det(A) = \lvert A \rvert = 
    \begin{vmatrix}
    a_{11}&a_{12}&\cdots&a_{1n}\\
    a_{21}&a_{22}&\cdots&a_{2n}\\
    \vdots&\vdots&&\vdots\\
    a_{m1}&a_{m2}&\cdots&a_{mn}\\
    \end{vmatrix}
$$
</p>

`余子式(cofactor)`：

<p>$$ M_{ij} $$</p>

`代数余子式`：

<p>$$ A_{ij} = (-1)^{i+j}det(A_{(ij)}) $$</p>

`非奇异矩阵`：

行列式不为0的矩阵

### 行列式性质

`行列式性质`：

- 行(列)互换，不影响行列式的值
- 如果某行(列)是其他行(列)的线性组合，则行列式的值为0
- 任意方阵A的行列式和他转置的行列式相同
- 单位矩阵的行列式为1： $ dat(I) = 1 $
- 一个Hermitian矩阵的行列式为实数：

<p>$$ det(A) = det(A^H) = det(A^T) \Rightarrow det(A) = det(A^{\ast}) = \lvert det(A) \rvert ^{\ast} $$</p>

- 两个矩阵乘积的行列式为他们行列式的乘积

<p>$$ det(AB) = det(A) det(B) \qquad A,B \in C^{m,n} $$</p>

- 对于三角矩阵，其行列式等于主对角线所有元素的乘积
- 对于任意常数(可以是复数) $ c $ ，有 $ det(cA) = c^n det(A) $
- 如果A是非奇异的，则 $ det(A^{-1}) = (det(A))^{-1} $
- 对于矩阵 $ A_{m \times m}, B_{m \times n}, C_{n \times m}, D_{n \times n} $ ，我们有：

<p>
$$
A非奇异 \Leftrightarrow det 
\begin{vmatrix}
A & B \\
C & D \\
\end{vmatrix} 
= det(A) det(D-CA^{-1}B) \\\\
D非奇异 \Leftrightarrow det 
\begin{vmatrix}
A & B \\
C & D \\
\end{vmatrix} 
= det(D) det(A-BD^{-1}C) \\
$$ 
</p>

### 行列式不等式

`行列式不等式`：

- Cauchy-Schwartz不等式：

<p>$$ \lvert det(A^HB) \rvert ^2 \leq det(A^HA)det(B^HB) $$</p>

- Hadamard不等式：对于 $ m \times m $ 矩阵A：

<p>$$ det(A) \leq \prod_{i=1}^{m}(\sum_{j=1}^{n} \lvert a_{ij} \rvert ^2) ^{1/2} $$</p>

- Fischer不等式：

<p>
$$
det(
\begin{bmatrix}
A & B \\
B^H & C \\
\end{bmatrix} 
) \leq det(A)det(C)
$$ 
</p>

- Minkowski 不等式：如果 $ A_{m \times m},B_{m \times m} \neq O_{m \times m} $ 且是半正定的，有：
  
<p>$$ \sqrt[m]{det(A+B)} \geq \sqrt[m]{det(A)} + \sqrt[m]{det(B)} $$</p> 

- 正定矩阵A的行列式大于0
- 半正定矩阵A的行列式大于或等于0
- 若 $ m \times m $ 矩阵A半正定，则

<p>$$ (det(A))^{1/m} \leq \frac{1}{m}det(A) $$</p>

- 若 $ A_{m \times m}, B_{m \times m} $ 均半正定
  
<p>$$ det(A+B) \geq det(A) + det(B) $$</p>

- 若 $ A_{m \times m} $ 正定， $ B_{m \times m} $ 半正定

<p>$$ det(A+B) \geq det(A) $$</p>

- 若 $ A_{m \times m} $ 正定， $ B_{m \times m} $ 半负定

<p>$$ det(A+B) \leq det(A) $$</p>


****
## 矩阵的秩

`矩阵的秩`：

矩阵 $ A_{m \times n} $ 的秩定义为该矩阵中线性无关的行(或列)的数目「行列是相等的」

对于矩阵方程 $ A_{m \times n}x_{n \times 1} = b_{m \times 1} $ ，我们根据m,n的大小，可以分为三类：

`适定方程`(well-determinded)：

$ m = n $ 且 $ rank(A) = n $ ，此时方程的解是唯一的，唯一解由 $ x = A^{-1}b $ 给出

`欠定方程`(under-determinded)：

$ m < rank(A) $ ，此时独立方程的个数小于独立的未知参数个数，即这样的方程组存在无穷多解

`超定方程`(over-determinded)：

$ m > rank(A) $ ，方程过剩，此时没有使得方程组严格满足的精确解 $ x $

<label class="label-theorem">秩的定义:</label>

矩阵 $ A_{m \times n} $ 的列空间 $ R(A) $ 的维数是该矩阵的秩：

<p>$$ r_A = dim[R(A)] $$</p>

由此我们可以得到几组等价的叙述：

- $ rank(A) = k $ 
- 存在A的k列且不多于k列组成的一线性无关组
- 存在A的k行且不多于k行组成的一线性无关组
- 存在A的一个 $ k \times k $ 子矩阵具有非零行列式，且所有 $ (k+1) \times (k+1) $ 子矩阵都有零行列式
- 列空间 $ R(A) $ 的维数为A
- $ k = n - dim[ Null(A) ] $ 其中 $ Null(A) $ 代表矩阵A的零空间

### 秩的几条定理

<p>$$ r(A) + r(B) -n  \leq r_{AB} \leq \min{\lbrace r_A,r_B \rbrace} $$</p>

<p>$$ rank(A+B) \leq rank[A,B]  \leq rank(A)+rank(B) $$</p>

<p>$$ rank(PA) = rank(AQ) = rank(A) ,\qquad 其中P满列秩,Q满行秩 $$</p>

### 秩的性质

`秩的性质`：

- 秩是一个正整数
- 秩等于或小于行数(或列数)
- 满秩(full rank)
- 秩亏缺(rank deficient)，此时为奇异矩阵
- 满行秩 (full row rank)
- 满列秩 (full column rank)
- 任何矩阵A左乘满列秩矩阵或右乘满行秩矩阵不改变其秩
- 对于 $ rank(A) = r $ 即我们存在一个 $ r \times r $ 的矩阵满秩

### 矩阵的等式

`矩阵的等式`：

- $ rank(A) = rank(A^H) = rank(A^T) = rank(A^{\ast}) $
- $ rank(cA) = rank(A) $
- 如果两方阵AC非奇异，则对于任意矩阵 $ B_{m \times n} $ 有 $ rank(AB) = rank(BC) = rank(ABC) = rank(B) $
- 如果两个同型矩阵有相同的秩，则存在两方阵 $ XY $ ，使得 $ B = XAY $
- $ rank(AA^T) = rank(A^TA) = rank(AA^H) = rank(A^HA) = rank(A) $
- 方阵A有 $ rank(A) = m \Leftrightarrow det(A) \neq 0 $
- $ rank \begin{bmatrix} A & B \\\\ C & D \end{bmatrix} = m \Leftrightarrow D = CA^{-1}B $

### 秩的不等式

`秩的不等式`：

- $ rank(A) \leq \min{\lbrace m,n \rbrace} $
- $ rank(A+B) \leq rank(A) + rank(B) $
- $ rank(A) + rank(B) -n  \leq rank(AB) \leq \min{\lbrace rank(A),rank(B) \rbrace} $ 
- 子矩阵的秩不大于原矩阵的秩