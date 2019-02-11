---
title: 矩阵分析与应用（九）：Moore-Penrose 逆矩阵
date: 2019-02-10 21:52:12
tags: Matrix
categories: Math # 分类
thumbnail: /assets/img/posts/Math/matrix.jpg
---

# Moore-Penrose 逆矩阵

我们之前一节的广义逆矩阵已经包含了逆矩阵，、左逆矩阵和右逆矩阵

我们现在面临一个问题：最小二乘解是非唯一的，其会有通解。这就会引申出两个问题————

- 是否存在某种意义下的唯一解？
- 若存在唯一解，那么广义逆矩阵 $ AGA = A $ 是否仍旧有效？

为此，我们引入了涵义更为广泛的广义逆矩阵：`Moore-Penrose 逆矩阵`

## Moore-Penrose 逆矩阵的定义和性质

我们用 $ P_s $ 表示到向量空间  $ S $ 上的正交投影，明显的，有 $ P_sX $ 在空间 $ S $ 上，且 $ x-P_sX $ 与子空间 $ S $ 正交。

对于任意一个 $ m \times n $ 的复矩阵 $ G $ 我们使用 $ Range(G) $ 来表示其值域空间。

Moore则证明出了「在1935年」，矩阵 $ G $ 的广义逆矩阵 $ G^{\dagger} $ 必须满足条件：

<p>
$$
GG^{\dagger} = P_{Range(G)},\qquad G^{\dagger}G = P_{Range(G^H)}
$$
</p>

我们将上述两条件称为`Moore条件`，而满足Moore条件的矩阵 $ G^{\dagger} $ 被称为`Moore逆矩阵`

此外，由于上述条件不好使用，我们有Penrose「在1955年」提出的另外一组条件

### Moore-Penrose条件

`Moore-Penrose条件`:

令A是任意的 $ m \times n $ 矩阵，若称矩阵 $ G $ 是 $ A $ 的广义逆矩阵，则需要满足下面四个条件：

- AGA = A
- GAG = G
- AG 为 Hermitian 矩阵「 $ (AG)^H = AG $ 」
- GA 为 Hermitian 矩阵「 $ (GA)^H = GA $ 」

### 以条件满足数目分类

- `自反广义逆矩阵`(reflexive generalized inverse):只满足条件(1)和(2)
  - 「也称为双条件广义逆矩阵或是 $ g_2 $ 广义逆矩阵」
- `正规化广义逆矩阵`(normalized generalized inverse):只满足条件(1)、(2)和(3)
  - 「也称为三条件广义逆矩阵或是 $ g_3 $ 广义逆矩阵」
- `弱广义逆矩阵`(weak generalized inverse)：只满足条件(1)、(2)和(4)
- `Moore-Penrose逆矩阵`:满足4个条件

### 一般广义矩阵的特性

- 若 $ A^g $ 是矩阵 $ A $ 的任意一种广义逆矩阵，则有：

<p>$$ rank(A^g) \geq rank(A) = rank(A^gA) = rank(AA^g) $$</p>

- 秩 $ rank(A^g) = rank(A) $ 的一个充要条件：$ A^g $ 是 $ A $ 的自反广义逆矩阵

我们上面的`Moore-Penrose条件`可以引申到下面的几条性质：

- $ (A^H) ^{\dagger} = (A^{\dagger}) ^H $
- $ A^{\dagger} AA^H = A^H AA^{\dagger} = A^H $
- $ AA^H (A^H) ^{\dagger} = (A^H) ^{\dagger} A^HA = A $
- $ AA^{\dagger} , A^{\dagger}A , (I-AA^{\dagger}) , (I-A^{\dagger}A) $ 均为幂等矩阵

### 之前几篇博客里的广义逆矩阵的特例

- $ n \times n $ 的正方非奇异矩阵 $ A $ 的逆矩阵 $ A^{-1} $ 满足4个条件
- $ m \times n $ 的 $ A_{m \times n} (m > n) $ 的左伪逆矩阵 $ (A^HA) ^{-1} A^H $ 满足4个条件
- $ m \times n $ 的 $ A_{m \times n} (m < n) $ 的右伪逆矩阵 $ A^H (AA^H) ^{-1} $ 满足4个条件
- 满足 $ LA_{m \times n} = I_n $ 的一般左逆矩阵 $ L_{n \times m} $ 满足 (1),(2)和(4)条件，是弱广义逆矩阵
- 满足 $ A_{m \times n}R = I_m $ 的一般右逆矩阵 $ R_{n \times m} $ 满足 (1),(2)和(3)条件，是正规化广义逆矩阵
- 广义逆矩阵 $ A^{-} $ 只满足条件 (1)

<label class="label-theorem"> 注意： </label>

不同于左逆矩阵 $ L $ 右逆矩阵 $ R $ 和广义逆矩阵 $ A^{-} $ 的多值性，Moore-Penrose逆矩阵定义唯一。

一般的，我们使用`广义逆矩阵`直接当作Moore-Penrose逆矩阵的简称「它包括了很多广义逆矩阵」，使用 $ A^{\dagger} $ 表示。

而原来的广义逆矩阵「即只用条件(1) $ AGA = A $ 定义的」广义逆矩阵只用 $ A^{-} $ 表示。

<label class="label-warning"> 警告： </label>

一般情况下， $ A^{\dagger} $ 并不满足逆矩阵的性质 $ (AB)^{-1} = B^{-1} A^{-1} $ ，即有：

<p>$$ (AB)^{\dagger} \neq B^{\dagger} A^{\dagger} $$</p>

### 满足 $ (AB)^{\dagger} \neq B^{\dagger} A^{\dagger} $ 的条件

若 $ A,B $ 均为使得矩阵 $ AB $ 存在的任意矩阵，则(AB)^{\dagger} = B^{\dagger} A^{\dagger} 的充要条件为以下之一：

- $ A^{\dagger}AB B^H A^H = BB^H A^H $ 和 $ BB^{\dagger} A^HAB = A^HAB $
- $ A^{\dagger} ABB^H $ 和 $ A^HA BB^{\dagger} $ 都是Heritian矩阵
- $ A^{\dagger}A BB^H A^HA BB^{\dagger} = BB^H A^HA $ 
- $ A^{\dagger}AB = B(AB)^{\dagger}AB $ 且 $ BB^{\dagger} A^H = A^HAB (AB)^{\dagger} $

我们确定任意一个 $ m \times n $ 的矩阵 $ A $  的 Moore-Pensore逆矩阵都可以由下面确定:

<p>$$ A^{\dagger} = (A^HA) ^{\dagger} A^H = A^H (AA^H) ^{\dagger} $$</p>

### Moore-Penrose逆矩阵 $ A^{\dagger} $ 的性质

- 广义逆矩阵 $ A^{\dagger} $ 唯一
- $ (A^H) ^{\dagger} = (A^{\dagger}) ^H = A^{\dagger H} = A^{H \dagger} $
- $ (A^{\dagger}) ^{\dagger} = A $
- 若 $ c \neq 0 $，则有 $ (cA)^{\dagger} = \frac{1}{c} A^{\dagger} $
- 若 $ D = diag(d_{11},d_{22},\cdots,d_{nn}) $ 为 $ n \times n $ 对角矩阵，则 $ D^{\dagger} = diag(d_{11}^{\dagger}, d_{22}^{\dagger}, \cdots, d_{nn}^{\dagger}) $ 「其中 $ d_{ii}^{\dagger} = d_{ii}^{-1} $ 或 $ d_{ii}^{\dagger} = 0 $ 」
- 零矩阵的广义逆矩阵为零矩阵，即： $ O_{m \times n}^{\dagger} = O_{n \times m} $
- 向量 $ x $ 的Moore-Penrose逆矩阵为： $ x^{\dagger} = (x^Hx) ^{-1} x^H $
- 关于几个真假的判断：
  - $ AA^{\dagger} \neq I_{m} $
  - $ A^{\dagger}A \neq I_{n} $
  - $ A^H (A^H) ^{\dagger} \neq I_{n} $
  - $ (A^H) ^{\dagger} A^H \neq I_{m} $
  - $ A^{\dagger}A A^H = A^H $
  - $ A^{H} AA^{\dagger} = A^{H} $
  - $ A^{H} AA^{\dagger} = A^{H} $
  - $ A^{H} A^{\dagger}A = A^{H} $
  - $ A A^{\dagger} (A^{\dagger}) ^H = (A^{\dagger}) ^H $
  - $ (A^{\dagger}) ^H   A^{\dagger} A = (A^{\dagger}) ^H $ `这里存疑。。。`
  - $ (A^H) ^{\dagger} A^H A = A $
  - $ A A^H (A^H) ^{\dagger} = A $
  - $ A^H (A^{\dagger}) ^H A^{\dagger} = A^{\dagger} $
  - $ A^{\dagger} (A^{\dagger}) ^H A^H = A^{\dagger} $
- 任何矩阵 $ A_{m \times n} $ 的广义逆矩阵都可以用 $ A^{\dagger} = (A^H A) ^{\dagger} A^H $ 或者 $ A^{\dagger} = A^H (AA^H) ^{\dagger} $ 确定，且他们有特殊情况：
  - 若 $ A $ 列满秩，则 $ A^{\dagger} = (A^HA) ^{-1} A^{\dagger} $ ，此时退化为左伪逆矩阵 
  - 若 $ A $ 行满秩，则 $ A^{\dagger} = A^H (AA^H) ^{-1} $ ，此时退化为右伪逆矩阵 
  - 若 $ A $ 为非奇异的正方矩阵，则 $ A^{\dagger} = A^{-1} $ ，此时退化为逆矩阵
- 若 $ A^H A = PDP^H $ ，其中 $ PP^H = P^HP = I $ ，且 $ D $ 为对角矩阵，则 $ A^{\dagger} = PD^{\dagger} P^H A^H $ 
- 若 $ A = BC $ ，且 $ B $ 列满秩， $ C $ 行满秩，则有：<p>$$ A^{\dagger} = C^{\dagger} B^{\dagger} = C^H (CC^H) ^{-1} (B^HB) ^{-1} B^H $$</p>
- 若 $ A^H = A $ ，其 $ A^2 = A $ ，则 $ A^{\dagger} = A $
- 如矩阵 $ A_{i} $ 相互正交， 即 $ A_i^H A_j = O $ ，则我们有：<p>$$ (A_1 + A_2 + \cdots + A_m) ^{\dagger} = A_1^{\dagger} + A_2^{\dagger} + \cdots + A_m^{\dagger} $$</p>
- $ (AA^H) ^{\dagger} = (A^{\dagger}) ^H A^{\dagger} $
- $ (AA^H) ^{\dagger} (AA^H) = AA^{\dagger} $
- 一般来说 $ (A^m) ^{\dagger} \neq (A^{\dagger}) ^m $ ，但只要 $ AA^H = A^HA $ ，则有 $ (A^m) ^{\dagger} = (A^{\dagger}) ^m $
- 若 $ A $ 为 $ m \times n $ 矩阵，则有：
    <p>
    $$
    \begin{bmatrix}
        A_{m \times n} & O_{m \times q} \\
        O_{p \times n} & O_{p \times q} \\
    \end{bmatrix} ^{\dagger} = 
    \begin{bmatrix}
        (A^{\dagger})_{n \times m} & O_{n \times p} \\
        O_{q \times m} & O_{q \times p} \\
    \end{bmatrix}
    $$
    </p>
    <p>
    $$
    \begin{bmatrix}
        O_{p \times q} & O_{p \times n} \\
        O_{m \times q} & A_{m \times n} \\
    \end{bmatrix} ^{\dagger} = 
    \begin{bmatrix}
        O_{q \times p} & O_{q \times m} \\
        O_{m \times q} & (A^{\dagger})_{n \times m} \\
    \end{bmatrix}
    $$
    </p>
- 对于广义逆矩阵的秩，有：
    <p>
    $$
    \begin{split}
        rank(A^{\dagger}) 
        & = rank(A) = rank(A^H) \\
        & = rank(A^{\dagger}A) = rank(AA^{\dagger}) \\
        & = rank(AA^{\dagger}A) = rank(A^{\dagger}AA^{\dagger}) \\
    \end{split}
    $$
    </p>
- 广义逆矩阵 $ A^{\dagger} $ 和 $ A^H $ 的行空间相同「即他们的行空间都互相包含」
- 广义逆矩阵 $ A^{\dagger} $ 和 $ A^H $ 的列空间相同「即 $ Span(A^{\dagger}) = Span(A^H) $ 或者 $ Range(A^{\dagger}) = Range(A^H) $ 」
- 对于 $ m > n $ ，且 $ rank(A) = n $ 时，我们有广义逆矩阵（左伪逆矩阵）$ A^{\dagger} = (A^HA) ^{-1} A^H $
  - $ A $ 和 $ AA^{\dagger} $ 的列空间相同
  - $ (I_m -AA^{\dagger}) $ 的列空间是矩阵 $ A $ 的列空间的正交补
  - $ AA^{\dagger} = A(A^HA) ^{-1} A^H $ 是幂等矩阵
  - $ I_m-AA^{\dagger} $ 是幂等矩阵
- 对于 $ m < n $ ，且 $ rank(A) = m $ 时，我们有广义逆矩阵（右伪逆矩阵）$ A^{\dagger} = A^H (AA^H) ^{-1} $
  - $ A^{\dagger} $ 和 $ A^{\dagger}A $ 的列空间相同
  - $ (I_n -A^{\dagger}A) $ 的列空间是矩阵 $ A^H $ 的列空间的正交补
  - $ A^{\dagger}A = A^H (AA^H) ^{-1} A $ 是幂等矩阵
  - $ I_n - A^{\dagger}A $ 是幂等矩阵
- 若 $ A_{m \times n} , B_{m \times p} $ ，则我们有:
    <p>
    $$
    \begin{bmatrix}
        A,& B \\
    \end{bmatrix} ^{\dagger} = 
    \begin{bmatrix}
        A^{\dagger} - A^{\dagger} B (C^{\dagger}+D) \\
        C^{\dagger}+D
    \end{bmatrix}
    $$
    </p>
    其中，我们有 $ C = (I_m - AA^{\dagger})B $ ，且 $ D = (I_p - C^{\dagger}C) [I_p + (I_p-C^{\dagger}C) B^H (A^{\dagger}) ^H B (I_p - C^{\dagger}C)] ^{-1} B^H (A^{\dagger}) ^H (I_m - BC^{\dagger}) $
- 若 $ A_{m \times n} , B_{p \times n} $ ，则我们有：
    <p>
    $$
    \begin{bmatrix}
        A \\
        B \\
    \end{bmatrix} ^{\dagger} = 
    \begin{bmatrix}
        A^{\dagger} - TBA^{\dagger}, &T
    \end{bmatrix}
    $$
    </p>
    其中，我们有 $ T = E^{\dagger} + (I_n -E^{\dagger}B) A^{\dagger} (A^{\dagger}) ^H B^H K(I_p-EE^{\dagger}) $ ，且 $ K = [I_p + (I_p - EE^{\dagger}) BA^{\dagger} (A^{\dagger}) ^H B^H (I_p - EE^{\dagger})]^{-1} $