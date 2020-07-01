---
title: IC-Reliability ：电子元器件可靠性
date: 2018-10-16 08:52:05
tags: 
- IC
- Reliability
categories: IC # 分类
thumbnail: /assets/img/posts/IC/Reliability/default.jpg
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")关于的IC学习记录

# 失效分布

## 浴盆曲线 Bathroom curve
- 曲线横轴
    + 早期失效 Infant Mortality 「老化期」
    + 随机失效 Nomal Life
    + 损耗 Wear-out Mortality
- 允许失效率 $ \lambda_0 $
- 使用寿命 

个别产品只有早期失效和损耗失效——设计工艺不合理

## 威布尔分布 Weibull distribution

研究链的强度

链强度概率分布即极小值分布问题

- pro
    + 应用广泛
    + 寿命试验数据处理
- con
    + 参数$ m, r, t_0 $ 比较复杂
    + 区间估计值过长
    + 采用概率纸估计法

累计失效函数:

\begin{align}
    F(t) & = 1 - e^{-\frac{(t-r)\^m}{t_0}} \qquad & (t >= r) \\\\
         & = 0 \qquad & (t < r)
\end{align}

真尺参数 $ \eta $ ，有 $ t_0 = \eta^m $ 