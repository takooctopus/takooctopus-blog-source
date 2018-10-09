---
title: LeetCode 每日算法 853 ：Car Fleet 
date: 2018-10-08 15:40:14
tags: LeetCode
categories: LeetCode # 分类
thumbnail: /assets/img/posts/LeetCode-Daily-Algorithm/default.jpg # 略缩图
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对[LeetCode](https://leetcode.com/ "LeetCode")每日打卡签到，做点记录。

# 853. Car Fleet 

# 题目描述

`N` cars are going to the same destination along a one lane road.  The destination is target miles away.

Each car `i` has a constant speed `speed[i]` (in miles per hour), and initial position `position[i]` miles towards the target along the road.

A car can never pass another car ahead of it, but it can catch up to it, and drive bumper to bumper at the same speed.

The distance between these two cars is ignored - they are assumed to have the same position.

A car fleet is some non-empty set of cars driving at the same position and same speed.  Note that a single car is also a car fleet.

If a car catches up to a car fleet right at the destination point, it will still be considered as one car fleet.


How many car fleets will arrive at the destination?

# Example 

|   Example 1:  |   |
|   :------ |   :------|
|Input: |   `target = 12`, `position = [10,8,0,5,3]`, `speed = [2,4,1,1,3]`   |
|Output:|    3  |

## Explanation: 

The cars starting at 10 and 8 become a fleet, meeting each other at 12.

The car starting at 0 doesn't catch up to any other car, so it is a fleet by itself.

The cars starting at 5 and 3 become a fleet, meeting each other at 6.

Note that no other cars meet these fleets before the destination, so the answer is 3.

# Note:

```c
0 <= N <= 10 ^ 4
0 < target <= 10 ^ 6
0 < speed[i] <= 10 ^ 6
0 <= position[i] < target
All initial positions are different.

```

# Solution

```python
class Solution:
    def carFleet(self, target, position, speed):
        """
        :type target: int
        :type position: List[int]
        :type speed: List[int]
        :rtype: int
        """  
        time = [float(target - p) / s for p, s in sorted(zip(pos, speed))]
        res = cur = 0
        for t in time[::-1]:
            if t > cur:
                res += 1
                cur = t
        return res         

```

实现不算难，主要是排序。

我们将每辆车到目的地的时间按照其到目标地点的距离大小排序「从大到小」。

然后我们对比每辆车的时间。

- 如果远的车比近的车所需时间大，即其会被认作一个单独的车。
- 如果较小，他们两车合并为一辆车


最终结果依次加一，返回就行了
