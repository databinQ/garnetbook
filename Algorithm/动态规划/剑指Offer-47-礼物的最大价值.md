# 题目描述

[剑指 Offer 47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)

在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

示例 1:
```
输入:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 12
解释: 路径 1→3→5→2→1 可以拿到最多价值的礼物
```

提示：

- 0 < grid.length <= 200
- 0 < grid[0].length <= 200

# 解题思路

## 动态规划

使用了滑动数组.

```python
class Solution:
    def maxValue(self, grid: List[List[int]]) -> int:
        n = len(grid)
        if not n:
            return 0
        m = len(grid[0])
        dp = [0]
        for v in grid[0]:
            dp.append(v + dp[-1])

        for i in range(1, n):
            for j in range(m):
                dp[j + 1] = max(dp[j], dp[j + 1]) + grid[i][j]
        return dp[-1]
```
