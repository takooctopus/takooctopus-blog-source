---
title: LeetCode 每日算法 789 ：Escape The Ghosts
date: 2018-10-09 20:25:15
tags: LeetCode
categories: LeetCode # 分类
thumbnail: /assets/img/posts/LeetCode-Daily-Algorithm/default.jpg # 略缩图
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对[LeetCode](https://leetcode.com/ "LeetCode")每日打卡签到，做点记录。

# 789. Escape The Ghosts

# 题目描述

You are playing a simplified Pacman game. You start at the point `(0, 0)`, and your destination is `(target[0], target[1])`. There are several ghosts on the map, the `i-th` ghost starts at `(ghosts[i][0], ghosts[i][1])`.

Each turn, you and all ghosts simultaneously *may* move in one of 4 cardinal directions: `north`, `east`, `west`, or `south`, going from the previous point to a new point 1 unit of distance away.

You escape if and only if you can reach the target before any ghost reaches you (for any given moves the ghosts may take.)  If you reach any square (including the target) at the same time as a ghost, it doesn't count as an escape.

Return True if and only if it is possible to escape.

# Example 

|   Example 1:|      |
|   :------ |   :------ |
|   Input:|   ghosts = [[1, 0], [0, 3]], target = [0, 1]   |
|   Output:|    true   |

Explanation: 
You can directly reach the destination (0, 1) at time 1, while the ghosts located at (1, 0) or (0, 3) have no way to catch up with you.

|   Example 2:|      |
|   :------ |   :------ |
|   Input:|   ghosts = [[1, 0]], target = [2, 0]  |
|   Output:|    false   |

Explanation: 
You need to reach the destination (2, 0), but the ghost at (1, 0) lies between you and the destination.

|   Example 3:|      |
|   :------ |   :------ |
|   Input:|    ghosts = [[2, 0]], target = [1, 0]   |
|   Output:|    false   |

Explanation: 
The ghost can reach the target at the same time as you.



# Note:

```c
All points have coordinates with absolute value <= 10000.
The number of ghosts will not exceed 100.
```

# Solution

```python
class Solution(object):
    def escapeGhosts(self, ghosts, target):
        """
        :type ghosts: List[List[int]]
        :type target: List[int]
        :rtype: bool
        """
        tx, ty = target
        d = abs(tx) + abs(ty)
        return all(d < abs(tx - i) + abs(ty - j) for i, j in ghosts)

```

这题很简单，直接绝对值就能结束，我就不贴我的答案了。

但是这里有一个`all()`函数，用作集合函数，`all()`只有全为真的时候才返回真。能够简略语句，记下了。

同理，`any()`是全为否时返回否，下面这个也能简略：

```python
class Solution:
    def escapeGhosts(self, ghosts, target):
        return not any([abs(ghost[0] - target[0]) + abs(ghost[1] - target[1]) <= abs(target[0]) + abs(target[1]) for ghost in ghosts])
```
