---
title: LeetCode 每日算法 258 ：Add Digits
date: 2018-10-09 17:15:24
tags: LeetCode
categories: LeetCode # 分类
thumbnail: /assets/img/posts/LeetCode-Daily-Algorithm/default.jpg # 略缩图
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对[LeetCode](https://leetcode.com/ "LeetCode")每日打卡签到，做点记录。

# 258. Add Digits 

# 题目描述

Given a non-negative integer num, repeatedly add all its digits until the result has only one digit.

# Example 

|   Example 1:  |   |
|   :------ |   :------|
|Input: |   `38`   |
|Output:|   `2` |

## Explanation: 

The process is like: `3 + 8 = 11`, `1 + 1 = 2`. Since `2` has only one digit, return it.

# Follow up:

Could you do it without any loop/recursion in O(1) runtime?


# Solution

```python
class Solution:
    def addDigits(self, num):
        """
        :type num: int
        :rtype: int
        """
        if num == 0:
            return 0
        t = num % 9
        return t if t else 9
        
```

我们主要要去考虑是否有O(1)的算法。

考虑题目假设有一个三位数`abc`，我们能够看见:

\begin{align} 
abc &= 100a + 10b + c \\\\
sum &= a + b + c
\end{align}

两式相减我们能够看见多余的部分

\begin{align}
minus = 99a + 9b
\end{align}

明显的，多余的部分为`9`的倍数。

我们可以以`9`取模。

但是在这其中有两个特例：
- 一个是`0`，因为所有`9`的倍数最终都模`0`，而大于`0`的数最终都不会得`0`。
- 同理是`9`，我们只需让所有模`9`后为`0`的返回`9`就行了。

