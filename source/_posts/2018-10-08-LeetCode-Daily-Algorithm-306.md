---
title: LeetCode 每日算法 306 ：Additive number
date: 2018-10-08 16:21:21
tags: 
- CS
- LeetCode
categories: CS # 分类
thumbnail: /assets/img/posts/CS/LeetCode.jpg # 略缩图
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对[LeetCode](https://leetcode.com/ "LeetCode")每日打卡签到，做点记录。

# 306. Additive number

# 问题描述

Additive number is a string whose digits can form additive sequence.

A valid additive sequence should contain at least three numbers. Except for the first two numbers, each subsequent number in the sequence must be the sum of the preceding two.

Given a string containing only digits `'0'-'9'`, write a function to determine if it's an additive number.

Note: Numbers in the additive sequence cannot have leading zeros, so sequence `1, 2, 03` or `1, 02, 3` is invalid.

# Example

|   Example 1:  |       |
|   :------ |   :------ |
|   Input:  |   "112358"  |
|   Output: |   true    |

Explanation: The digits can form an additive sequence: `1, 1, 2, 3, 5, 8`. 
`1 + 1 = 2`, `1 + 2 = 3`, `2 + 3 = 5`, `3 + 5 = 8`

|   Example 2:|     |
|   :------ |   :------ |
|   Input:  |    "199100199"    |
|   Output: |    true   |

Explanation: The additive sequence is: `1, 99, 100, 199`. 
`1 + 99 = 100`, `99 + 100 = 199`

## Follow up:

How would you handle overflow for very large input integers?

# Solution:

```python
class Solution:
    def isAdditiveNumber(self, num):
        """
        :type num: str
        :rtype: bool
        """
        
        n = lenghth(num);
        for i,j in itertools.combinations(range(1, n), 2):
            a, b = num[:i], num[i:j]
            if b != str(int(b)):
                continue
            while j < n:
                c = str(int(a) + int(b))
                if not num.startswith(c, j)
                    break
                j += len(c)
                a, b = b, c
            if j == n:
                return True
        return False
```

我们在这里反向地运用比较会比较方便

其中`itertools.combinations(a, b)`会返回所有长度为`b`的组合「序列」，其顺序按照原来排序。`startswith(a,b)`很简单地看`a`是不是以`b`开头。

最核心的一点是在比较第三个数时，我们直接采用前两个数的和，转化为字符串进行比较，可以有效的规避首位`0`带来的问题。同理我们直接比较`b`转化`int`再转化的`str`，可以规避第一次首位`0`问题。

最终只要位指针指向了末尾，我们就能够判断其是不是Additive Number了。
