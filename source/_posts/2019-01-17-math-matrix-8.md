---
title: 矩阵分析与应用（八）：广义逆矩阵
date: 2019-01-18 16:23:15
tags:  Matrix
categories: Math # 分类
thumbnail: /assets/img/posts/Math/matrix.jpg
---

# 广义逆矩阵

广义逆矩阵就需要将逆矩阵的概念推广到长方形矩阵或者奇异的正方矩阵中，用`对线性最小二乘法`进行统一的理论解释。

## 左逆矩阵和右逆矩阵

只要一个矩阵 $ L $ 和矩阵 $ A $ 的乘积是单位矩阵，即 $ LA = I $ ，即它就是逆矩阵，其中有三种可能：

- L存在，且唯一
- L存在，但不唯一
- L不存在

而逆矩阵的形式也有3种：

- 满足 $ AA^{-1} = A^{-1}A = I $ 的逆矩阵 $ A^{-1} $
- 只满足 $ LA = I $ 的 $ L $ ,称为`左逆矩阵`
- 只满足 $ AR = I $ 的 $ R $ 称为`右逆矩阵`

<label class = "label-theorem">定理：</label>

仅当 $ m \geq n $ 时，矩阵 $ A \in C^{m \times n} $ 可能有左逆矩阵
仅当 $ m \leq n $ 时，矩阵 $ A \in C^{m \times n} $ 可能有右逆矩阵

### 单边逆矩阵的唯一解

`左伪逆矩阵(left pseudo inverse)`

如果 $ m > n $ 且 A 满列秩「rank(A)=n」时，有唯一左逆矩阵 $ L $ ，称为`左伪逆矩阵`。此时有

<p>$$ L = (A^HA) ^{-1} A^H $$</p>

`右伪逆矩阵(right pseudo inverse)`

同理，如果 $ m < n $ 且 A 满行秩「rank(A)=m」时，有唯一右逆矩阵 $ R $ ，称为`右伪逆矩阵`。此时有

<p>$$ R = A^H (A^HA) ^{-1}  $$</p>

<label class="label-example">应用</label>

左伪逆矩阵和超定方程的最小二乘解密切相关

右伪逆矩阵和欠定方程的最小二乘最小范数解密切联系

****
## 广义逆矩阵的定义和性质

### 一致性方程(consistent equation)

`一致性方程`：

若矩阵A行之间存在的线性关系同时也存在于向量y的对应元素之间，则称 $ A_{m \times n}x_{n \times 1} = y_{m \times 1} $ 为`一致性方程`

当且仅当方程为一致性方程时，这线性方程组可以求解

`检验一致性方程`：

线性方程 $ Ax=y $ 是一致的，当且仅当增广矩阵 $ [A,y] $ 的秩等于矩阵A 的秩，即：

<p>$$ rank([A,y]) = rank(A) $$</p>

### 广义逆矩阵G(generalized inverse)

`广义逆矩阵G`

若A是一个 $ m \times n $ 矩阵，且具有任意秩。即矩阵A的广义逆矩阵是一个 $ n \times m $ 矩阵 $ G $ ，并且使得当 $ Ax = y $ 为一致性方程时，$ x = Gy $ 是线性方程的解。

一致性方程 $ Ax = y $ 对于 $ y \neq 0 $ 时有解 $ x = Gy $ ，当且仅当 $ AGA = A $

<label class="label-theorem"> 命题 </label>

方程 $ Ax = 0 $ 的解与矩阵A的任意行正交，并且线性无关。

「我们知道这一定是一个一致性方程，即矩阵A之中行之间的关系存在于0向量中。线性方程也一定是有解的。用 $ a^T $ 表示矩阵中的任意一行，$ \tilde{x} $ 表示方程的一个解，即有 $ a^T \tilde{x} = 0 $ ，即解与A中任意一行正交。」

<label class='label-warning'>符号</label>

$ m \times n $ 矩阵A的广义逆矩阵G用符号 $ A^- $ 表示，即 $ G = A^- $

### 广义逆矩阵的性质

$ A^- $ 存在 $ \Leftrightarrow AA^-A=A $ 

<label class="label-example">证明</label>

#### $ \Rightarrow $ 的证明

令 $ y = Az $ 且 $ z $ 是一个 $ n \times 1 $ 的任意向量，即有 $ Ax = y $ 是一致性方程。

在这里，广义逆矩阵 $ A^- $ 存在的话，就意味着：

<p>$$ A(A^-Az) = A(A^- y) = Az , \quad \forall z \quad \Rightarrow AA^-A=A $$</p>

#### $ \Leftarrow $ 的证明

若 $ AGA = A $ ，我们需要证明G就是矩阵A的广义逆矩阵 $ A^- $

若 $ Ax = y $ 是一致性方程，则 $ \exists $ 解向量 $ w $ 满足 $ Aw = y $ 。

由于 $ AGA = A $ ，即 $ AGAw = Aw \Rightarrow AGy = Aw = y $ 。即我们看到 $ Gy $ 满足线性方程 $ Ax = y $

即 $ Gy $ 是 $ Ax = y $ 的一个解向量，即 $ G = A^- $

### 引理

<label class="label-theorem">引理：</label>

- $ A^- $ 存在 $ \Leftrightarrow H = A^-A $ 为幂等矩阵「就是 $ H^2 = H $ 」 且 $ rank(H) = rank(A) $
- $ A^- $ 存在 $ \Leftrightarrow F = AA^- $ 为幂等矩阵「就是 $ F^2 = F $ 」 且 $ rank(F) = rank(A) $

这个我们用上面的 $ AA^-A = A $ 同时右乘一个 $ A $ 即可证明 $ H^2 = H $

而矩阵性质： $ rank(AB) \leq rank(A) $ 或者 $ rank(AB) \leq rank(B) $ ，

又有 $ H = A^-A $ 以及 $ AH = AA^-A = A $

即我们有：$ rank(A) \geq rank(H) \geq rank(AH) \geq rank(A) $

得证 $ rank(H) = rank(A) $

而对于 $ \Leftarrow $ 的证明：

我们假定 $ H = A^-A $ 是幂等矩阵，且 $ rank(H) =rank(A) $ 

即我们有 $ H(I-H) = O \Rightarrow A^- A(I-A^-A) = O \Rightarrow A(I-A^-A) = O \Rightarrow AA^-A = A $

### 广义逆矩阵的其他两种定义

#### 定义一：

$ m \times n $ 矩阵A的广义逆矩阵是一个满足 $ AA^-A = A $ 的 $ n \times m $ 矩阵 $ A^- $

#### 定义二：

$ m \times n $ 矩阵A的广义逆矩阵是满足下列两个条件之一的 $ n \times m $ 的矩阵 $ A^- $

- $ A^-A $ 为幂等矩阵，且 $ rank(A^-A) = rank(A) $
- $ AA^- $ 为幂等矩阵，且 $ rank(AA^-) = rank(A) $

<label class="label-example">验证：</label>

若矩阵 $ A_{m \times n} $ 有一个主子矩阵 $ A_{11} $ 且其秩 $ r = rank(A) $ ,且A的分块形式为：

<p>
$$
A = 
\begin{bmatrix}
    A_{11} & A_{12} \\
    A_{21} & A_{22} \\
\end{bmatrix}
, \qquad 且
A_{22} = A_{21}A^{-1}_{11}A_{12}
$$
</p>

则其广义逆矩阵 $ A^{-} $ 为：

<p>
$$
A^{-} = 
\begin{bmatrix}
    A_{11}^{-1} & O \\
    O & O \\
\end{bmatrix}
$$
</p>

****
## 广义逆矩阵的计算

### 满秩分解

`满秩分解(full-rank decomposition)`:

令 $ A_{m \times n} $ 具有秩r。将其分解为 $ A = FG $ ，其中 $ F_{m \times r} $ 和 $ G_{r \times n} $ 均具有秩r，则称这是矩阵的`满秩分解`。

我们可以通过矩阵的相似对角化去证明出来。

为此我们得到了`满秩分解算法`：

- 利用初等行变换将矩阵 $ A $ 化为阶梯形：
  
  <p>
  $$
    \begin{bmatrix} 
    G_{r \times n} \\
    O_{(m-r) \times n} \\  
    \end{bmatrix} 
  $$
  </p>

- 对单位矩阵 $ I $ 进行第一步的逆初等行变换得到 $ P^{-1} $
- 利用 $ P^{-1} $ 的前r列构造矩阵 $ F $
- 书写满秩分解结果 $ A = FG $

#### 引理: $ A^- $ 的性质

若矩阵 $ A_{m \times n} $ 具有秩r，且其满秩分解为 $ A = F_{m \times r}G_{r \times n} $，则我们知道其广义逆矩阵为：

<p>$$ A^- = G^T(F^TAG^T)^{-1}F^T $$</p>

证明也很简单，带入 $ AA^-A = A $ 就能证明了。

### 广义逆矩阵的计算

- 假设 $ A_{m \times n} $ ， 且 $ u_{m \times 1 } v_{n \times 1} $ 是两个一维向量，则有：
  
    <p>
    $$
    (A + uv^T)^- = A^- - \frac{(A^-u)(u^TA^-)}{1 + u^TA^-u}
    $$
    </p>

- 分块矩阵的广义逆矩阵计算公式：
    
    若
    <p>
    $$
    M = 
    \begin{bmatrix}
        A & C \\
        C^H & B \\    
    \end{bmatrix}
    $$
    </p>

    其中 $ A = X^H_1X_1 $ ，$ B = X^H_2X_2 $ ，$ C = X^H_1X_2 $ ，若设 $ D = B - C^H A^-C $ ，则我们有 $ M^- $ ：

    <p>
    $$
    M^- = 
    \begin{bmatrix}
        A^- + A^-CD^-C^HA^- & -A^-CD^- \\
        -D^-C^HA^- & D^- \\    
    \end{bmatrix}
    $$
    </p>

- 矩阵之和的广义逆矩阵的计算公式：

    若 $ AA^-UBV = UBV $「即UBV的列空间是A的列空间的子集」 与 $ UBVA^-A = UBV $ 「即UBV的行空间是A的行空间的子集」，则我们有 $ G = A + UBV $ 的广义逆矩阵 $ G^- $ 存在几种求法：

    <p>
    $$
    \begin{split}
        G^-_1 &= A^- -A^-(A^- + A^-UBVA^-)^-A^-UBVA^- \\
        G^-_2 &= A^- -A^-U(U + UBVA^-U)^-UBVA^- \\
        G^-_3 &= A^- -A^-UB(B + BVA^-UB)^-BVA^- \\
        G^-_4 &= A^- -A^-UBV(V + VA^-UBV)^-VA^- \\
        G^-_5 &= A^- -A^-UBVA^-(A^- + A^-UBVA^-)^-A^- \\
    \end{split}
    $$
    </p>

    <label class = "label-example"> 解释：</label>
    
    也许上面的不容易看清，但是规律是相同的。括号中的第一位是括号两边的相同字母，然后第二位是由括号右边和括号左边组合而成的「中间的 $ A^- $ 少了一个作为连接」

****
## 一致方程的最小范数解

### 通解

若 $ n \times m $ 矩阵 $ A^- $ 是　$ A_{m \times n} $ 的任意一个广义逆矩阵，则有：

- 齐次方程 $ Ax = 0 $ 的一个通解是 $ x = (I-A^-A)z $ ，其中 $ z $ 是任意的 $ n \times 1 $ 的向量「很容易证明吧，和上面一样」
- 非齐次方程 $ Ax = y $ 为一致方程的充要条件为：

    <p>
    $$
    AA^-y = y
    $$
    </p>

- 非齐次方程 $ Ax = y $ 的一个通解为：

    <p>
    $$
    x = A^-y + (I-A^-A)z
    $$
    </p>

### 最小范数解

`最小范数条件`

<p>$$ \min_{Ax = y} \lVert x \rVert = \lVert Gy \rVert $$</p>

此时称矩阵 $ G $ 为`最小范数广义逆矩阵`

### 伴随矩阵（区别于常规的伴随矩阵）:

为此我们定义 $ A_{m \times n} $ 伴随矩阵的符号为 $ A_{n \times m}^{\sharp} s $ ，且有两向量 $ x_{n \times 1}，y_{m \times 1} $ ，而我们定义将m阶向量空间的内积等价变换为n阶向量的内积的一个映射：

<p>$$ \langle Ax,y \rangle _m = \langle x,A^{\sharp}y \rangle _n $$</p>

<label class="label-warning">警告：</label>

这里的伴随矩阵是之前我们说的「比如在逆矩阵一节里那个」 $ \overset{adj(A)}{伴随矩阵} $ ，但是表示方式略有不同。

此外如果 $ A^{\sharp} = A $ ，我们称其为`自伴随矩阵`。「当然，我们一般更熟悉他的另一个名字`Hermitian matrix`，就是厄米特矩阵」

在此，还有些性质：

- $ (A^{\sharp}) ^{\sharp} = A $
- $ (AB)^{\sharp} = B^{\sharp} A^{\sharp} $
- $ \langle Ax,By \rangle , \forall x,y \Leftrightarrow A^{\sharp}B = 0 $
- $ A^{\sharp} = A^T $ 「 $ A $ 为实矩阵」或 $ A^{\sharp} = A^H $ 「 $ A $ 为复矩阵」

### 最小范数解的求取

若 $ Gy $ 是一致方程 $ Ax = y $ 的最小范数解，当且仅当：

<p>$$ AGA=A ,\quad (GA)^{\sharp} = GA $$</p>

前一个条件很容易就能看出来，是定义所决定的。

至于第二个条件，我们已经知道通解是 $ x = A^-y + (I-A^-A)z $， 即 $ x = Gy + (I-GA)z $ ，我们只需证明：

<p>$$ \lVert Gy \rVert \leq \lVert Gy + (I-GA)z \rVert , \quad \forall z $$</p>

或者：

<p>
$$
\begin{split}
& \lVert GAb \rVert \leq \lVert GAb + (I-GA)z \rVert , \quad \forall b,z \\
\Leftrightarrow & \langle GAb,(I-GA)z \rangle = 0 , \quad \forall b,z \\
\Leftrightarrow & (GA)^{\sharp}(I-GA) = O \\
\Leftrightarrow & (GA)^{\sharp} = (GA)^{\sharp}GA \\
\end{split}
$$
</p>

因为我们最后要得到 $ (GA)^{\sharp} = GA $ ，即我们易知：

<p>$$ (GA)^{\sharp}GA = GAGA = GA = (GA)^{\sharp}  $$</p>

而要是不相同，很容易就能用反证法得出结果。

#### 注释

<label class = "label-theorem">注释：</label>

我们这里关于最小范数解还有两点需要强调的：

- 充要条件 $ AGA = A , \quad (GA)^{\sharp} = GA $ ，我们能够写成`等价形式` $ GAA^{\sharp} =  A^{\sharp} $
- 令 $ G_1,G_2 $ 是矩阵 $ A $ 的两个不同的广义逆矩阵，由上得知 $ G_iAA^{\sharp} = A^{\sharp} $，即有：
    <p>
    $$
    (G_1-G_2)AA^{\sharp} = O \Leftrightarrow (G_1-G_2)AA^{\sharp} = O \Leftrightarrow G_1A = G_2A
    $$
    </p>

    而我们知道 $ Ax = y $ 是一致方程，即有 $ rank([A, y]) = rank(A) $ ，我们因此可以将 $ y $ 写作 $ Ab $ ，即有：
    
     <p>
    $$
    G_1Ab=G_2Ab \Rightarrow G_1y = G_2y
    $$
    </p>

    我们可以看到最小范数解是`唯一的`。

<label class = "label-example">特别的：</label>

我们讨论 $ A_{m \times n} $ 具有满行秩m时，线性方程 $ Ax = y $ 的最小范数解。

我们知道 A 满行秩，即是有增广矩阵 $ rank([A, y]) = rank(A) $ ，即线性方程 $ Ax = y $ 是一致方程。此外，又因为矩阵乘积 $ AA^H $ 可逆，故存在右伪逆矩阵 $ A^H (A A^H) ^{-1} $

即我们与之对应的解为：

<p>$$ x^{\circ} = A^H(AA^H)^{-1}y $$</p>

但它是否是最小范数解呢？

我们简单的证明一下：

假设 $ x $ 是不同的任意解，则有：

<p>$$ \lVert x \rVert ^2 = \lVert x^{\circ} + x - x^{\circ}  \rVert ^2  = \lVert x^{\circ} \rVert ^2 + \lVert x -x^{\circ} \rVert ^2 + 2(x^{\circ})^H(x-x^{\circ}) $$</p>

带入 $ x^{\circ} = A^H(A A^H) ^{-1}y = A^H(A A^H) ^{-1}Ax $ 的值，我们得到：

<p>
$$
\begin{split}
    (x^{\circ})^H(x-x^{\circ}) 
    &= y^H(AA^H)^{-1}A [I-A^H(AA^H)^{-1}A]x \\
    &= y^H[(AA^H)^{-1}A-(AA^H)^{-1}A]x
    &= 0
\end{split} 
$$
</p>

即我们可以化简得到：

<p>$$  \lVert x \rVert ^2 = \lVert x^{\circ} \rVert ^2 + \lVert x -x^{\circ} \rVert ^2  $$</p>

由于向量范数的非负性，我们得到：

<p>$$ \lVert x \rVert ^2 \geq \lVert x^{\circ} \rVert ^2 $$</p>

即 $ x^{\circ} $ 确实为最小范数解。

#### 右伪逆矩阵满足最小范数解

右伪逆矩阵 $ G = A^{H} (AA^H) ^{-1} $ 满足最小范数解的条件 $ AGA = A, \quad (GA)^{\sharp} = GA $

用伴随矩阵特性 $ B^{\sharp} = B^H $ 就能证明

****
## 非一致方程的最小二乘解

对于非一致方程，其没有严格满足方程的解，即期只能有`近似解`。即我们需要寻找一个使得方程两边的误差平方和最小的解。我们称这个解为`非一致方程的最小二乘解`。

我们使用 $ \hat{x} $ 表示最小二乘解。

而它满足条件：

<p>$$ \lVert A\hat{x}-y \rVert = \inf_{x} \lVert Ax-y \rVert $$</p>

我们用 $ \inf $ 表示函数的下确界

### 最小二乘解的条件

令 $ G $ 为某个矩阵，要使得 $ \hat{x} = Gy $ 是非一致方程 $ Ax = y $ 的最小二乘解，`当且仅当`：

<p>$$ A^{\sharp}AG = A^{\sharp} $$</p>

或者等价于：

<p>$$ AGA = A, \quad (AG)^{\sharp} = AG $$</p>

<label class = "label-warning"> 我们注意其与上面所讲的一致方程的最小范数解之间的区别 </label>

为此，我们对这个也给予证明：

我们已知前提：

<p>$$ \lVert A\hat{x} - y \rVert \leq \lVert Ax - y \rVert , \quad \forall x,y $$</p>

而带入 $ \hat{x} = Gy $

我们有：

<p>
$$
\begin{split}
    \lVert AGy - y \rVert 
    &\leq \lVert Ax - y \rVert , \quad \forall x,y\\
    &\leq \lVert AGy - y + Aw \rVert , \quad \forall x,w = x - Gy \\
    &\Leftrightarrow \langle Aw,(AG-I)y \rangle = 0 , \quad \forall y,w \\
    &\Leftrightarrow A^{\sharp}(AG-I) = O \\
    &\Leftrightarrow A^{\sharp}AG = A^{\sharp}
\end{split}
$$
</p>

我们看得出来，这个证明过程和之前的一致方程的最小范数解的证明很相似。

- 上面两边同时右乘 $ A $ ，即有：

    <p>$$ A^{\sharp}(AGA) = A^{\sharp}A $$</p>

    要使得对所有矩阵 $ A $ 都存在，即我们有：

    <p>$$ AGA = A $$</p>

- 上面两边同时左乘矩阵 $ G^{\sharp} $ ，我们能够得到：

    <p>$$ G^{\sharp}A^{\sharp}AG = (AG)^{\sharp}AG = (AG)^{\sharp} $$</p>

    我们可以使用之前的方式证明其充要条件是：

    <p>$$ (AG)^{\sharp} = AG $$</p>

#### 注释

<label class = "label-theorem">注释：</label>

- 非一致方程的最小二乘解有`可能不是唯一的`，但是不同的最小二乘解得到的 $ Ax $ 和 $ Ax - y $ 是唯一的。
- 非一致方程的最小二乘解的`通解形式`为:

    <p>$$ \hat{x} = Gy + (I-GA)z , \quad \forall z $$</p>


<label class = "label-example">特殊情况(满列秩)：</label>

当非一致方程 $ Ax = y $ 的矩阵 $ A $ 有满列秩的特殊情况，此时 $ A^HA $ 显然是非奇异的。

而此时解：

<p>$$ x^{\circ} = (A^HA)^{-1}A^Hy $$</p>

是一个`最小二乘解`

#### 左伪逆矩阵满足最小二乘解

左伪逆矩阵 $ G = (AA^H) ^{-1} A^{H} $ 满足最小二乘解的条件 $ AGA = A, \quad (AG)^{\sharp} = AG $

用伴随矩阵特性 $ B^{\sharp} = B^H $ 就能证明