---
title: PHPSTORM Tips (一)
date: 2017-03-21 16:24:09
tags: PHPSTORM
categories: PHP # 分类
thumbnail: /assets/img/posts/PHPSTORM-TIPS/1.jpg
---
# 用PHPSTORM连接数据库
大家接触php也有一段日子了，一般来说，
最开始大家都会推荐使用像是xampp和wamp这样的集成环境，
这些会默认提供数据库管理软件MySQLAdmin

{% asset_img xampp1.PNG %}

但我们是有追求的，所以我们选择使用phpstorm来进行更为方便快捷的管理
首先我们来找到右侧的datab+ase选项
如果你不小心关掉了， 那可以点击View-Tool Window-Database来开启它

{% asset_img phpstorm1.PNG %}

之后我们就能在右侧看见database的选项了

{% asset_img phpstorm2.PNG %}

点击+号，选择Data Source 再选择 MySQL

{% asset_img phpstorm3.PNG %}

我们能便能打开一个设置窗口

{% asset_img phpstorm4.PNG %}

先别急，选择左侧的MySQL选项，在Dialect中选择MySQL并选择版本，等一会让它下载完成后就能切回去了，在之前的页面填好你的数据库信息
测试一下，点击OK
我们就能在右侧看见自己的数据库了

{% asset_img phpstorm5.PNG %}