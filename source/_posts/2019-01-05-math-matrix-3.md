---
title: 矩阵分析与应用（三）：随机向量
date: 2019-01-05 16:57:24
tags: Matrix
categories: Math # 分类
thumbnail: /assets/img/posts/Math/matrix.png
---


# 随机向量

这节我们讨论概率：

概率论基本概念：

- 基本事件： $ \omega $
- 事件：$ A (\in \mathcal{F}) $
- 事件的全体： $ \mathcal{F} $
- 事件的概率: $ P(A) $

## $ L^2 $ 理论 

对于概率空间：$ (\Omega, \mathcal{F}, P) $ ，我们以 $ L_p = L_p(\Omega,\mathcal{F},P) $ 表示随机变量 $ \xi = \xi (\omega) $ 的空间

`Banach空间`：

我们定义满足如下的式子的 $ L_P (P>1) $ 为Banach空间：

<p>$$ E \lbrace \lvert \xi \rvert ^P \rbrace  = \int_{\Omega} \lvert \xi \rvert ^P dp \qquad < \infty $$</p>

$ L_2 $ `空间`：

在Banach空间中，起重要作用的是空间 $ L_2 =  L_2(\Omega,\mathcal{F},P) $ 

这种空间有限二阶矩 $ E \lbrace \xi ^2 \rbrace < \infty $

我们称其为 $ L_2 $ 空间

$ L^2 $ `理论`：

只讨论向量空间中一阶和二阶的统计性质的理论


****

## 概率密度函数

一个含有 m 个随机变量的实值向量：

<p>$$ x(\xi) = [x_1(\xi), x_2(\xi), \cdots, x_m(\xi)] $$</p>

- `联合累积分布函数`：
    <p>$$ F(x) = F_x(x_1,x_2,\cdots,x_m) $$</p>
- `联合概率密度函数`：
    <p>
    $$ 
    \begin{split}
    f(x)    &= f_x(x_1,x_2,\cdots,x_m) \\
            &= \frac{\partial ^m}{\partial x_1 \partial x_2 \cdots \partial x_m}F_x(x_1, x_2, \cdots , x_m)
    \end{split}
    $$
    </p>
- `边缘概率密度函数`
    <p>
    $$ 
    f(x_i) \overset{def}{=} \int_{-\infty}^{\infty} \cdots \int_{-\infty}^{\infty} f_x(x_1,x_2, \cdots ,x_m)dx_1 \cdots dx_{i-1}dx_{x+1} \cdots dx_m
    $$
    </p>

### 联合独立

`联合独立`：

对随机变量 $ x_1(\xi), x_2(\xi), \cdots , x_m(\xi)$ 如果存在对这m个事件 $ \lbrace x_1(\xi) \leq x_1\rbrace , \lbrace x_2(\xi) \leq x_2\rbrace , \cdots , \lbrace x_m(\xi) \leq x_m\rbrace $ 有:

<p>$$ P \lbrace x_1(\xi) \leq x_1 , \cdots , x_m(\xi) \leq x_m \rbrace = P \lbrace x_1(\xi) \leq x_1 \rbrace \cdots P \lbrace x_m(\xi) \leq x_m \rbrace$$</p>

即我们知道联合概率分布函数(联合概率密度函数)为各个边缘概率分布函数(边缘概率密度函数)之积，即称为联合独立

****
## 复随机向量的概率密度函数

与上面实随机向量相似，但在这里面需要加上复数域：

- `复随机向量`：
    <p>$$ x(\xi) = x_R(\xi) + jx_I(\xi) $$</p>
- `联合累积分布函数`：
    <p>$$ F(x) \overset{def}{=} P \lbrace x(\xi) \leq x \rbrace \overset{def}{=} P \lbrace x_R(\xi) \leq x_R , x_I(\xi) \leq x_I \rbrace $$</p>
- `联合概率密度函数`：
    <p>
    $$ 
    f(x) = \frac{\partial ^2m}{\partial x_{R1} \partial x_{I1} \partial x_{R2} \partial x_{I2} \cdots \partial x_{Rm} \partial x_{Im} }F_x(x_1, x_2, \cdots , x_m)
    $$
    </p>

<label class="label-warning"> 特别的： <label>

<p>$$ \int_{-\infty}^{\infty} f(x)dx = 1 $$</p>

****
## 随机向量的统计描述

### 均值向量

`均值向量`：

随机向量的数学期望就是均值向量：

<p>
$$
\mu _x = E \lbrace x(\xi) \rbrace = 
    \begin{bmatrix}
        E \lbrace x_1(\xi) \rbrace \\
        E \lbrace x_2(\xi) \rbrace \\
        \cdots \\
        E \lbrace x_m(\xi) \rbrace \\
    \end{bmatrix}
    =
    \begin{bmatrix}
        \mu _1 \\
        \mu _2 \\
        \cdots \\
        \mu _m \\
    \end{bmatrix}
$$
</p>

其中数学期望定义为：

<p>$$ E \lbrace x(\xi) \rbrace \overset{def}{=} \int_{-\infty}^{\infty} xf(x)dx $$</p>

我们很容易就能发现，均值向量的实质是随机向量的`一阶矩`

### 相关矩阵和协方差矩阵

随机向量的`二阶矩`是矩阵，其描述随机向量分布的散布情况：

#### 自相关矩阵

`自相关矩阵`：

<p>
$$
    R_x \overset{def}{=}    E \lbrace x(\xi)x^H(\xi) \rbrace = 
        \begin{bmatrix}
            r_{11}&r_{12}&\cdots&r_{1m}\\
            r_{21}&r_{22}&\cdots&r_{2m}\\
            \vdots&\vdots&&\vdots\\
            r_{m1}&r_{m2}&\cdots&r_{mmhu}\\
        \end{bmatrix}
$$
</p>

明显的，自相关矩阵是`共轭对称`的

`自相关函数`：

在上面矩阵中的主对角线元素就是自相关函数

<p>$$ r_{ii} \overset{def}{=}   E \lbrace \lvert x_i(\xi)^2 \rvert \rbrace  ,   i = 1,2,\cdots ,m $$</p>

`互相关函数`：

在上面矩阵中非主对角线的其他元素

<p>$$ r_{ij} \overset{def}{=}   E \lbrace x_i(\xi) x_j^{\ast}(\xi) \rbrace  ,   i,j = 1,2,\cdots ,m \qquad i \neq j $$</p>

#### 协方差矩阵

`自协方差矩阵`：

我们就可以得到我们随机向量 $ x(\xi) $ 的自协方差矩阵：

<p>
$$
    C_x \overset{def}{=}    E \lbrace [x(\xi)-\mu_x][x(\xi)-\mu_x]^H \rbrace = 
        \begin{bmatrix}
            c_{11}&c_{12}&\cdots&c_{1m}\\
            c_{21}&c_{22}&\cdots&c_{2m}\\
            \vdots&\vdots&&\vdots\\
            c_{m1}&c_{m2}&\cdots&c_{mm}\\
        \end{bmatrix}
$$
</p>

`方差`：

协方差矩阵主对角线上元素称为方差 $ \delta^2_i = c_{ii} $ 

`协方差`：

协方差矩阵非主对角线上元素表示随机变量 $ x_i(\xi) $ 和 $ x_j(\xi) $ 之间的协方差

<p>$$ c_{ij} \overset{def}{=}   E \lbrace [x_i(\xi) -\mu_i] [x_j(\xi) - \mu_j]^{\ast} \rbrace  =  E \lbrace x_i(\xi) x_j^{\ast}(\xi) \rbrace - \mu_i \mu_j ^{\ast} = c_{ji}^{\ast} $$</p>

<label class = "label-theorem"> 两矩阵关系 </label>

<p>$$ C_x = R_x -\mu_x \mu_x^H $$</p>


### 互相关矩阵和互协方差矩阵

`互相关矩阵`：

同理，我们也能得到两个随机向量 $ x(\xi) $ 和 $ y(\xi) $ 的互相关矩阵：

<p>
$$
    R_{xy} \overset{def}{=}    E \lbrace x(\xi)y^H(\xi) \rbrace = 
        \begin{bmatrix}
            r_{x_1,y_1}&r_{x_1,y_2}&\cdots&r_{x_1,y_m}\\
            r_{x_2,y_1}&r_{x_2,y_2}&\cdots&r_{x_2,y_m}\\
            \vdots&\vdots&&\vdots\\
            r_{x_m,y_1}&r_{x_m,y_2}&\cdots&r_{x_m,y_m}\\
        \end{bmatrix}
$$
</p>

`互协方差矩阵`：

<p>
$$
    C_xy \overset{def}{=}    E \lbrace [x(\xi)-\mu_x][y(\xi)-\mu_y]^H \rbrace = R_{xy} - \mu_x \mu_y^H =
        \begin{bmatrix}
            c_{x_1,y_1}&c_{x_1,y_2}&\cdots&c_{x_1,y_m}\\
            c_{x_2,y_1}&c_{x_2,y_2}&\cdots&c_{x_2,y_m}\\
            \vdots&\vdots&&\vdots\\
            c_{x_m,y_1}&c_{x_m,y_2}&\cdots&c_{x_m,y_m}\\
        \end{bmatrix}
$$
</p>

### 两个随机向量的统计不相关和正交

我们知道随机信号减去其自己的均值之后，只剩下了随机变化的部分，即互协方差表示了 $ x_i(\xi) $ 和 $ x_j(\xi) $ 之间`随机变化部分`的相乘

即这个可以提取出两个信号随机变化之间的共性，并且抑制非共性部分

即互协方差表现了 $ x_i(\xi) $ 和 $ x_j(\xi) $ 之间的`相关联程度`

<label class = "label-warning"> 互协方差越大，则相关程度越强 </label>

`相关系数`：

两个随机变量 $ x(\xi) $ 和 $ y(\xi) $ 之间的相关系数定义：

<p>$$ \rho_{xy} \overset{def}{=} \frac{c_{xy}}{\sqrt{ E \lbrace \lvert x(\xi)^2 \rvert \rbrace  E \lbrace \lvert y(\xi)^2 \rvert \rbrace}} = \frac{c_{xy}}{\delta_x \delta_y } $$</p>

明显的，由Cauchy-Schwartz不等式知道：

<p>$$ 0 \leq \lvert \rho_{xy} \rvert \leq 1 $$</p>

<label class = "label-warning"> $ \rho_{xy} $ 越大，则相关程度越强 </label>

`统计不相关`：

两个随机向量 $ x(\xi) $ 和 $ y(\xi) $ 统计不相关，即它们的互协方差矩阵为零矩阵 $ C_{xy} = O $，即有：

<p>$$ r_{xy} = E \lbrace x(\xi)y^{\ast}(\xi) \rbrace = 0 $$</p>

`正交`：

两个随机向量 $ x(\xi) $ 和 $ y(\xi) $ 互相关矩阵为零矩阵，即  $ R_{xy} = O $ ，则称其为正交

### 随机向量的线性变换

在这里我们对复正态分布的随机向量的线性变换的一些性质做点分析：

对于复常数矩阵A，和复正态分布随机向量 $ x(\xi) \sim CN(\mu_x , \Gamma_x) $ 有线性变换：

<p>$$ y(\xi) = Ax(\xi) $$</p>

明显的，y仍然为正态随机向量

`均值`：

<p>$$ \mu_y = E \lbrace y(\xi) \rbrace = E \lbrace Ax(\xi) \rbrace = AE \lbrace x(\xi) \rbrace = A \mu_x $$</p>

`自相关矩阵`：

<p>$$ R_y = E \lbrace y(\xi)y(\xi)^H \rbrace = E \lbrace Ax(\xi)x(\xi)^HA^H \rbrace = AE \lbrace x(\xi)x(\xi)^H \rbrace A^H = AR_xA^H $$</p>

`自协方差矩阵`：

同理，我们能得到：

<p>$$ C_y = AC_xA^H $$</p>

`互相关矩阵`：

<p>$$ R_{xy} = R_{x}A^H $$</p>

<label class = "label-warning"> 顺序改变后有些许变化：</label>

<p>$$ R_{yx} = R_{xy}^H = AR_{x} $$</p>

`互协方差矩阵`：

<p>$$ C_{xy} = C_{x}A^H $$</p>

<p>$$ C_{yx} = C_{xy}^H = AC_{x} $$</p>


****
## 正态随机向量

### 实正态随机向量

`实正态随机向量`：

我们定义一个均值向量为 $ \mu_x $ 协方差矩阵为 $ \Gamma_x $ 的 实正态随机向量记作 $ x \sim N(\mu_x , \Gamma_x) $

此时其概率密度函数：

<p>$$ f(x) = \frac{1}{(2\pi)^{m/2} { \lvert \Gamma_x \rvert }^{1/2} }  exp[-\frac{1}{2}(x-\mu_x)^T \Gamma_x^{-1} (x-\mu_x) ] $$</p>

`正定二次型`：

<p>$$ (x-\mu_x)^T \Gamma_x^{-1} (x-\mu_x) = \sum_{i=1}^{m} \sum_{j=1}^{m} \Gamma_x^{-1} (i,j) (x_i-\mu_i) (x_j-\mu_j) $$</p>

`特征函数`：

<p>$$ \Phi_x(\omega) = exp(j\omega^T\mu_x - \frac{1}{2} \omega^T \Gamma_x \omega) $$</p>

### 复正态随机向量

`复正态分布随机向量`：

我们记作 $ x \sim CN(\mu_x , \Gamma_x) $

`概率密度函数`：

假设其各个变量统计独立，有

<p>$$ f(x) = \frac{1}{\pi^m \lvert \Gamma_x \rvert}    exp[-(x-\mu_x)^T \Gamma_x^{-1} (x-\mu_x) ] $$</p>

`特征函数`：

<p>$$ \Phi_x(\omega) = exp[jRe(\omega^T\mu_x) - \frac{1}{4} \omega^T \Gamma_x \omega] $$</p>

<label class = "label-warning"> 对于x的线性相关 $ y(\xi) = Ax(\xi) $ </label>

其实概率密度函数和复概率密度函数均和上面的`相同`


