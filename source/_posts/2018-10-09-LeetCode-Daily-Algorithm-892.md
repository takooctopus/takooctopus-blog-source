---
title: LeetCode 每日算法 892 ：Surface Area of 3D Shapes
date: 2018-10-09 17:53:45
tags: LeetCode
categories: LeetCode # 分类
thumbnail: /assets/img/posts/LeetCode-Daily-Algorithm/default.jpg # 略缩图
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对[LeetCode](https://leetcode.com/ "LeetCode")每日打卡签到，做点记录。

# 892. Surface Area of 3D Shapes 

# 题目描述

On a `N * N` grid, we place some `1 * 1 * 1` cubes.

Each value `v = grid[i][j]` represents a tower of `v` cubes placed on top of grid cell `(i, j)`.

Return the total surface area of the resulting shapes.

# Example 

|   Example 1:|      |
|   :------ |   :------ |
|   Input:|    [[2]]   |
|   Output:|    10   |

|   Example 2:|      |
|   :------ |   :------ |
|   Input:|    [[1,2],[3,4]]   |
|   Output:|    34   |

|   Example 3:|      |
|   :------ |   :------ |
|   Input:|    [[1,0],[0,2]]   |
|   Output:|    16   |

|   Example 4:|      |
|   :------ |   :------ |
|   Input:|    [[1,1,1],[1,0,1],[1,1,1]]   |
|   Output:|    32   |

|   Example 5:|      |
|   :------ |   :------ |
|   Input:|    [[2,2,2],[2,1,2],[2,2,2]]   |
|   Output:|    46   |


# Note:

```c
1 <= N <= 50
0 <= grid[i][j] <= 50
```

# Solution


我做的时候比较麻烦，速度比较慢。
```python
class Solution:
    def surfaceArea(self, grid):
        """
        :type grid: List[List[int]]
        :rtype: int
        """
        n = len(grid)
        s = 0
        y = 0
        z = 0
        for i in range(n):
            for j in range(n):
                for kx, ky in ((i-1, j), (i+1, j), (i, j-1), (i, j+1)):
                    if kx == -1 or ky == -1 or kx == n or ky == n:
                        s_m = 0
                    else:
                        s_m = grid[kx][ky]
                    s += min(s_m, grid[i][j])
                y += grid[i][j]
                if grid[i][j]:
                    z += 1
        return 2 * z + 4 * y - s
```
我做的速度比较慢

```python
class Solution:
    def surfaceArea(self, grid):
        n, res = len(grid), 0
        for i in range(n):
            for j in range(n):
                if grid[i][j]: res += 2 + grid[i][j] * 4
                if i: res -= min(grid[i][j], grid[i - 1][j]) * 2
                if j: res -= min(grid[i][j], grid[i][j - 1]) * 2
        return res
```

看其他人的，大约比我快了12ms，我看了其中最为主要的一点是他在加边时只选了上左两边，相比于我将4个方向全加了能省下一半的工作，看起来我的考虑不够周到。

```python
class Solution:
    def surfaceArea(self, grid):
        return sum(v * 4 + 2 for row in grid for v in row if v) - sum(min(a, b) * 2 for row in grid + list(zip(*grid)) for a, b in list(zip(row, row[1:])))        
```

这个一句话的程序...

只能说原来可以这样简化句子，这个一句话程序和我的思想差不多，主要是注意`2 for row in grid for v in row if v`这样的简化思想。

