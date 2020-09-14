# 解题思路

[剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

给定一棵二叉搜索树，请找出其中第k大的节点。

示例 1:
```
输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 4
```

示例 2:
```
输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 4
```

限制：

- 1 ≤ k ≤ 二叉搜索树元素个数

# 解题思路

二叉搜索树的中序遍历是递增数组, 如果我们将左右遍历的顺序调换, 先遍历右子树, 再遍历左子树, 得到的就是递减的数组. 使用递归的方法, 对遇到的有效结点计数, 遇到第`k`个时一路向上返回.


```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def kthLargest(self, root: TreeNode, k: int) -> int:
        count = 0
        def dfs(node):
            if node is None:
                return False, None

            state, val = dfs(node.right)
            if state:
                return state, val

            nonlocal count
            count += 1
            if count == k:
                return True, node.val
            return dfs(node.left)

        return dfs(root)[1]
```
