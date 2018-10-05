---
title: 'python 内核分析（一）：Python源码获取与基本情况'
date: 2018-09-25 13:37:52
tags: PYTHON
categories: PYTHON # 分类
thumbnail: /assets/img/posts/PYTHON-KERNEL/1.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# Python源码获取与基本情况

## 源码获取
```language-console
git clone https://github.com/python/cpython.git
```
## 版本选择与切换
```language-console
git branch -a
```

此时我们能够看到所有的版本
```language-console
* master
  remotes/origin/2.7
  remotes/origin/3.4
  remotes/origin/3.5
  remotes/origin/3.6
  remotes/origin/3.7
  remotes/origin/HEAD -> origin/master
  remotes/origin/backport-5bdab64-3.7
  remotes/origin/master
  remotes/origin/ncoghlan-bpo-33499-whats-new
  remotes/origin/pr_4170
 ```

 我们看到最新的稳定版是3.7，我们选择将版本选择到最新的稳定版
 ```language-console
git checkout 3.7
 ```

## 目录结构
首先我们看到的是目录结构:
  - Include：包括了Python提供的所有头文件，用于使用C或者C++来编写Python的自定义模块
  - Lib：Python的标准库，全部由Python编写
  - Module：包含用C语言编写的模块，比如常用的`random`以及`stringIO`等
  - Parses：包含了python解释器中的scanner和parser部分,也就是词法分析和语法分析部分,一个类似yacc一样根据规则自动生成
  - Object：包含所有Python的内置对象,整数, list, dict等.也包含了运行时python需要的所有内部使用的对象的实现
  - Python：包含了python解释器中Compiler和执行引擎部分,是python运行的核心所在
  - PCBuild：包含了vs工程文件

此外还有几个文件夹:
  - Doc
  - Grammar
  - m4
  - Mac
  - Misc
  - PC
  - Programs
  - Tools

## 总体架构
Python整体分成了3个模块:
  - 内建模块：比如我们引入的`os`就是内建模块
  - Python内核：定义Python的对象与类型系统以及其垃圾处理机制
  - Python虚拟机：解释器，对其代码进行语法分析
