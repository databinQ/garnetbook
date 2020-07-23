# 题目描述

[剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

```
[["a","b","c","e"],
["s","f","c","s"],
["a","d","e","e"]]
```

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

示例 1：
```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

示例 2：
```
输入：board = [["a","b"],["c","d"]], word = "abcd"
输出：false
```

提示：

- 1 <= board.length <= 200
- 1 <= board[i].length <= 200

# 解题思路

这种在二维矩阵中, 四个方向都可以走的题目, 基本都是通过**DFS**的方法解决, 每道题中需要注意的是**剪枝**的条件和**达成目标停止的条件**.

这道题中, 可以从任意一个位置出发, 所以我们要遍历所有位置, 作为起始点, 判断以这个点位起始点能不能完成目标. 其中一个符合则整体符合.

考虑其中一个初始位置. 我们在一个位置可以朝四个方向递归探索, 比较当前位置的字符与word中指定位置的字符是否相等, 如果相等, 才能继续向后比较. 这里有一点, 如果一个路径中包含`word`, 假设这个路径不以`word`开头, 将前面的子路径去掉, 剩余的路径以`word`开头, 也是符合要求的. 因此我们比较时, 就以`word`的首字符开始.

遇到以下情况剪枝/停止:

- 索引超越边界, 剪枝
- 当前位置的字符与word中指定位置的字符不等, 没有必要继续比较, 剪枝
- word最后一个字符已经比较完毕, 且相等, 已经找到, 返回成功, 停止

```python
class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        n, m, l = len(board), len(board[0]), len(word)
        def dfs(i, j, k):
            if not 0 <= i < n or not 0 <= j < m or board[i][j] != word[k]:  # 剪枝
                return False
            if k == l - 1:  # 最后一个字符比较完毕, 整体都符合
                return True

            t, board[i][j] = board[i][j], '&'  # 使用`&`, 表示这个位置已经遍历过, 因为word中不会出现这个符号, 递归遇到这个符号时, 肯定不相等, 第一时间剪枝
            res = dfs(i - 1, j, k + 1) or dfs(i + 1, j, k + 1) or dfs(i, j - 1, k + 1) or dfs(i, j + 1, k + 1)  # 上下左右
            board[i][j] = t  # 回溯, 恢复现场
            return res

        for i in range(n):
            for j in range(m):
                if dfs(i, j, 0):
                    return True
        return False
```
