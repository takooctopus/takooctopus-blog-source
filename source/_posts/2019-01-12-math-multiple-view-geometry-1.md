---
title: 多视图几何算法（一）：前言
date: 2019-01-12 17:43:32
tags: Multiple-View
categories: Math # 分类
thumbnail: /assets/img/posts/Math/multiple-view.jpg
---

# 前言

在过去的十年里，对于多视图几何算法的研究有了长足的进步，其理论和实践已经成熟到能够在十年前未解决的问题上有良好的表现，这些问题包括很多，我们列出几类：

- 给出两张图片，没有其他的信息。计算两张图片的匹配程度以及其中各点以及相机的3D位置。
- 给出三张照片，没有其他的信息。计算图片中点之间和线之间的匹配。并求出这些点、线和相机的位置。
- 在不需要校准参照物的情况下，计算 $ \overset{stereo \ rig}{立体装置} $ 的 $ \overset{Epipolar \  geometry}{对极几何} $ ，以及 $ \overset{trinocular \ rig}{三目装置} $ 的 $ \overset{trifocal \ geometry}{三焦几何} $
- 通过一个自然场景的图片序列计算出相机的 $ \overset{internal \ calibration}{内部校准} $ 

这些算法独特在于他们都是`未校准的`。

支撑这些算法的是对于多视图几何算法理论的更新更完备的理解：包括参数的数量、图片中点线之间的约束、以及对于相机和图片关联中的三维空间点的检索。「举例来说，计算立体装置的对极几何时，我们仅需要7个参数，其中不包括对相机的校准。我们可以通过7或以上的图片点的关联(而10年前的校准方法需要每个相机至少11个参数)」

上面的例子说明了`未校准方式(即投影)`的优越性——算法合适的表述会让那些每步计算都需要的参数变得明晰。避免了没用参数的运算。

在更为广泛的应用中，要一次校准相机然后不变是不现实的：比方说相机会移动「在移动设备上」，相机内部参数会改变「相机的伸缩焦距」。再甚至于有些情况下校准信息并不可得「比方说视频序列中计算相机移动或者从档案影片脚本中建立虚拟现实模型时，移动和内部校准信息都是未知的」

多视图几何算法的成就不仅仅是由于我们理论理解的提升，更是由于估算图像中数学物体的能力提升。

第一个提升就是我们意识到了在`超定系统`中误差需要让其尽量小「不论是代数的、几何的还是统计的」。第二个是使用`鲁棒性`方法「比方说RANSAC」，这样估计就不会受到数据集中的异常值影响。

许多关于重建的问题我们已经可以说它被解决了：

- 图像点关联的多焦点张量的估计「特别是基本矩阵和3焦点张量（4焦点的没有这么受关注）」
- 从这些张量中对于相机矩阵的提取「以及随后的从2、3、4个视图中的重建操作」

还有很多虽有一些进展，但仍旧亟需解决的问题：

- 对 $ \overset{bundle \ adjustment}{光束法平差} $ 的应用以适用于更一般的问题
- `公制(欧几里得)重建`在相机矩阵上有极小的假设
- 对于图像序列的自动检测以及对于异常值和错误值得清洗使用了`多焦点张量关系`

我们之后可以看到很多方面：

- 背景的 $ \overset{Homography}{单应性} $ 
- 单视图的 $ \overset{Camera \  matrix}{相机矩阵} $ 
- 双视图的 $ \overset{Fundamental \  matrix}{基础矩阵} $ 
- 三视图的 $ \overset{Trifocal \  tensor}{三焦向量} $ 
- 四视图的 $ \overset{Quadrifocal \  tensor}{四焦向量} $ 

引言就截止与此了，仅仅是简要的介绍了一下发展和基本的应用而已，更多的再由以后再看。

## 单词

maturity 成熟
[Epipolar geometry 对极几何](https://en.wikipedia.org/wiki/Epipolar_geometry) 
stereo rig 立体装置
trifocal 三焦
trinocular rig 三目装置
calibration 校准
distinctive 独特的
flavour 味道
underpin 支撑
retrieval 取回、找回、检索
correspondence 关联
explicit 明白
ambiguity 多义性
circumstance 情形
archive 档案
[overdeterminded system 超定系统](https://en.wikipedia.org/wiki/Overdetermined_system) 
outlier 异常值
multifocal tensor 多焦点张量
extraction 提取
subsequent 随后的
bundle 捆、束
[bundle adjustment 光束法平差、束调整、捆绑调整「鬼知道怎么翻译的」](https://en.wikipedia.org/wiki/Bundle_adjustment) 
matrices matrix的复数
metric 公制
elimination 清除、淘汰
[Homography 单应性](https://en.wikipedia.org/wiki/Homography_(computer_vision)) 
[Camera matrix 相机矩阵](https://en.wikipedia.org/wiki/Camera_matrix) 
[Fundamental matrix 基础矩阵](https://en.wikipedia.org/wiki/Fundamental_matrix_(computer_vision)) 
[Trifocal tensor 三焦向量](https://en.wikipedia.org/wiki/Trifocal_tensor) 
[Quadrifocal tensor 四焦向量「其实没有链接」](#) 