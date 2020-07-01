---
title: LeetCode 每日算法 829 ：Consecutive Numbers Sum
date: 2018-10-09 20:58:15
tags: 
- CS
- LeetCode
categories: CS # 分类
thumbnail: /assets/img/posts/CS/LeetCode.jpg # 略缩图
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对[LeetCode](https://leetcode.com/ "LeetCode")每日打卡签到，做点记录。

# 829. Consecutive Numbers Sum

# 题目描述

Given a positive integer N, how many ways can we write it as a sum of consecutive positive integers?

# Example 

|   Example 1:|      |
|   :------ |   :------ |
|   Input:|   5  |
|   Output:|    2   |

Explanation: `5 = 5 = 2 + 3`

|   Example 2:|      |
|   :------ |   :------ |
|   Input:|   9  |
|   Output:|   3   |

Explanation: `9 = 9 = 4 + 5 = 2 + 3 + 4`

|   Example 3:|      |
|   :------ |   :------ |
|   Input:|    15   |
|   Output:|    4   |

Explanation: `15 = 15 = 8 + 7 = 4 + 5 + 6 = 1 + 2 + 3 + 4 + 5`


# Note:

```c
 1 <= N <= 10 ^ 9.
```

# Solution

```python
class Solution(object):
    def consecutiveNumbersSum(self, N):
        """
        :type N: int
        :rtype: int
        """
        cnt=0
        for d in range(1, N+1):
            diff = d * (d-1)/2
            nd = N - diff
            if nd <= 0: break
            if nd % d == 0:
                cnt += 1

        return cnt
```
题设为

\begin{align}
N = n + n+1 + n+2 + ... + n+d-1
\end{align}

我们合并同类项

\begin{align}
N = n * d + \frac{d * (d-1)}{2}
\end{align}

转化即

\begin{align}
n * d = N - \frac{d * (d-1)}{2}
\end{align}

只要 左边能被d整除，我们就能判定这个成立.