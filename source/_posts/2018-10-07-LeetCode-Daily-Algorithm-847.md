---
title: LeetCode 每日算法 847 ：Shortest Path Visiting All Nodes 
date: 2018-10-07 22:42:20
tags: 
- CS
- LeetCode
categories: CS # 分类
thumbnail: /assets/img/posts/CS/LeetCode.jpg # 略缩图
---

>本文由[本人@takooctopus](https://takooctopus.github.io "たこ焼きのGITHUB")对[LeetCode](https://leetcode.com/ "LeetCode")每日打卡签到，做点记录。

# 847. Shortest Path Visiting All Nodes 


## 题目描述:
An undirected, connected graph of N nodes (labeled 0, 1, 2, ..., N-1) is given as graph.

`graph.length = N`, and `j != i` is in the list graph[i] exactly once, if and only if nodes i and j are connected.

Return the length of the shortest path that visits every node. You may start and stop at any node, you may revisit nodes multiple times, and you may reuse edges.

## Example 

|   Example 1:  |       |
|   :------ |   :-------    |
|   Input: |    [[1,2,3],[0],[0],[0]]    |
|   Output:|     4   |
|   Explanation:|    One possible path is [1,0,2,0,3]   |

|   Example 2: |        |
|   :------ |   :-------    |
|   Input:|  [[1],[0,2,4],[1,3,4],[2],[1,2]]  |
|   Output:|     4   |
|   Explanation:|    One possible path is [0,1,4,2,3]   |
 

## Note:

```c
1 <= graph.length <= 12
0 <= graph[i].length < graph.lengthi

```

# Solution

## Heapq Solution

```python
class Solution:
    def shortestPathLength(self, graph):
        memo, final, q = set(), (1 << len(graph)) - 1, [(0, i, 1 << i) for i in range(len(graph))]
        while q:
            steps, node, state = heapq.heappop(q)
            if state == final: return steps
            for v in graph[node]:
                if (state | 1 << v, v) not in memo:
                    heapq.heappush(q, (steps + 1, v, state | 1 << v))
                    memo.add((state | 1 << v, v))
```

上面是堆排序的方法：

其中`memo`为一个空集合，`final`的值为$ 2^{N}-1 $，即两点路径总数，`q`作为一个列表，保存了`[(0,0,2^0), ..., (0,N,2^N)]`共N+1个元素。注意`q`中元素实际上在循环内用作`[steps, node, state]`表示了其各个Node的状态。

`heappop()`返回最小的元素。

我们考虑二级的for循环：

其中`state`实际使用了二进制，对于节点`i`，其只有第`i`位为1。首先其状态`state`与`1 << v`抑或，将这条路径中加入新的节点`v`。

再将这一步后新产生的`(steps+1, 新节点, 新路径)`压入堆中。

再加入集合`memo`表示已经算过这条路径。

直到将所有点的与其他点的关系全部算清。

然后我们根据`steps`的数值从小到大依此取出列表`q`的元素。因为我们要将其全部走一遍，所以我们最终需要的是状态在2进制下N位全为一，换成10进制就是$ 2^{N}-1 $，即我们定义的`final`的值。

即当我们状态`states == final`时，我们就能得到最短步数了。


## BFS Solution

```python
class Solution:
    def shortestPathLength(self, graph):
        memo, final, q, steps = set(), (1 << len(graph)) - 1, [(i, 1 << i) for i in range(len(graph))], 0
        while True:
            new = []
            for node, state in q:
                if state == final: return steps
                for v in graph[node]:
                    if (state | 1 << v, v) not in memo:
                        new.append((v, state | 1 << v))
                        memo.add((state | 1 << v, v))
            q = new
            steps += 1
```

上面的相比之下为广度优先的搜索：

判断的原理和堆排差不多，`q`依旧表示`[node, state]`，但这次每走一步都会留下这一步的快照，并将其看作下一步开始状态。

因为是按照步数走，所以只要走完所有点，就一定为最小步数。


## Deque Solution

```python

class Solution:
    def shortestPathLength(self, graph):
        memo, final, q, steps = set(), (1 << len(graph)) - 1, collections.deque([(i, 0, 1 << i) for i in range(len(graph))]), 0
        while q:
            node, steps, state = q.popleft()
            if state == final: return steps
            for v in graph[node]:
                if (state | 1 << v, v) not in memo:
                    q.append((v, steps + 1, state | 1 << v))
                    memo.add((state | 1 << v, v))
```

上面是队列搜索：

在这里使用了`collection.deque()`，也能通过`appendleft()`从左侧添加。同理也有`pop()`以及`poplect()`。

我们最开始选择初始点，将其从`collection`左侧依次取出，当`(更新的状态, 要去的节点)`不重复时，将`(要去的节点，steps, 更新的状态)`新加进集合的右侧。

循环往复，也是将`steps`从小到大搜索，直到我们最终发现状态`state`更新为全部走过。
