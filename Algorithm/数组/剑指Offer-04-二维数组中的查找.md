# 题目描述

[剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

示例:
```
现有矩阵 matrix 如下：

[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```

给定 target = 5，返回 true。

给定 target = 20，返回 false。

限制：

- 0 <= n <= 1000

- 0 <= m <= 1000

# 解题思路

像题目中的排序二维数组, 搜索一般从**左下角**或者**右上角**开始. 如果从左上角开始, 向下向右搜索, 某个位置右面和下面都是不小于它的数字, 无法确定前进方向. 但如果是从左下角开始搜索, 当前位置大于目标值则向上, 小于目标值则向右.

其实观察某个位置, **小于它**的值全部在矩阵的**左上角**, 沿着这个边界, 向右上移动就好. 下图是`target = 8`的边界.

![](/Algorithm/imgs/378_fig2.png)

```python
class Solution:
    def findNumberIn2DArray(self, matrix: List[List[int]], target: int) -> bool:
        if not matrix:
            return False
        n, m = len(matrix), len(matrix[0])
        if m == 0:
            return False
        i, j = n - 1, 0
        while i > -1 and j < m:
            if matrix[i][j] == target:
                return True
            elif matrix[i][j] > target:
                i -= 1
            else:
                j += 1
        return False
```
