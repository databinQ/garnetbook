# 题目描述

[剑指 Offer 13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

示例 1：
```
输入：m = 2, n = 3, k = 1
输出：3
```

示例 2：
```
输入：m = 3, n = 1, k = 0
输出：1
```

提示：

- 1 <= n,m <= 100
- 0 <= k <= 20

# 解题思路

标准的DFS, 向四个方式搜索, 注意剪枝条件即可. 使用哈希表存储已经走过的格子, 配合剪枝, 以及完成总数量的统计.

```python
class Solution:
    def movingCount(self, m: int, n: int, k: int) -> int:
        seen = set()
        def dfs(i, j):
            if (i, j) in seen:
                return
            if not 0 <= i < m or not 0 <= j < n:
                return
            total = 0
            ti, tj = i, j
            while ti:
                total += ti - (ti // 10) * 10 if ti > 9 else ti
                ti = ti // 10
            while tj:
                total += tj - (tj // 10) * 10 if tj > 9 else tj
                tj = tj // 10
            if total > k:
                return

            seen.add((i, j))
            dfs(i - 1, j)
            dfs(i + 1, j)
            dfs(i, j - 1)
            dfs(i, j + 1)

        dfs(0, 0)
        return len(seen)
```
