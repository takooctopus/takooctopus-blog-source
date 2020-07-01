---
title: ThinkPHP 技巧 (一)
date: 2017-03-21 15:48:43
tags: PHP
categories: CS # 分类
thumbnail: /assets/img/posts/CS/THINKPHP.jpg
---
# ThinkPHP二级目录的配置
在最近的thinkphp安装时，出现了小的问题
如果将其放入服务器的二级文件夹下，会出现重定向和U方法跳转时不能到达正确的位置
比如出现只会跳转至服务器主目录下面产生404error错误
其实这个只需要一点小的更改就能够配好
我们打开我们的thinkphp文件夹
找到入口文件index.php
```php
<?php
// 检测PHP环境
if(version_compare(PHP_VERSION,'5.3.0','<'))  die('require PHP > 5.3.0 !');
// 开启调试模式 建议开发阶段开启 部署阶段注释或者设为false
define('APP_DEBUG',True);
// 定义应用目录
define('APP_PATH','./Application/');
// 引入ThinkPHP入口文件
require_once './ThinkPHP/ThinkPHP.php';
```
在其中增加
```php
//二级目录配置
define('THINK_PATH', '../PROJECT_ROUTE/ThinkPHP/');
define('APP_NAME','PROJECT_NAME');
```
再修改
```php
// 引入ThinkPHP入口文件
require_once THINK_PATH.'ThinkPHP.php';
```
如下面
```php
<?php
// 检测PHP环境
if(version_compare(PHP_VERSION,'5.3.0','<'))  die('require PHP > 5.3.0 !');
//二级目录配置
define('THINK_PATH', '../PROJECT_ROUTE/ThinkPHP/');
// 开启调试模式 建议开发阶段开启 部署阶段注释或者设为false
define('APP_DEBUG',True);
// 定义应用目录
define('APP_PATH','./Application/');
define('APP_NAME','PROJECT_NAME');
// 引入ThinkPHP入口文件
require_once THINK_PATH.'ThinkPHP.php';
```
就能好好的将其放入二级目录中了