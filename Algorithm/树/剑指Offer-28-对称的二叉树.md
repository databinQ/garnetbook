# 题目描述

[剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。

例如，二叉树 [1,2,2,3,4,4,3] 是对称的。
```
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

但是下面这个 [1,2,2,null,3,null,3] 则不是镜像对称的:
```
    1
   / \
  2   2
   \   \
   3    3
```

示例 1：
```
输入：root = [1,2,2,3,4,4,3]
输出：true
```

示例 2：
```
输入：root = [1,2,2,null,3,null,3]
输出：false
```

限制：

- 0 <= 节点个数 <= 1000

# 解题思路

迭代的方法, 使用两个栈(或队列, 分别对应DFS和BFS), 分别存储左子树和右子树中的所有结点. 左右子树结点压入栈中的顺序相反, 若左子树先压左节点, 右子树则先压右结点. 这样压入的结点, 按顺序, 在位置上是对称的. 剩余的只是要比较位置对称的结点, 值是否相等.

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def isSymmetric(self, root: TreeNode) -> bool:
        if root is None:
            return True
        stack_left, stack_right = [root.left], [root.right]
        while stack_left and stack_right:
            left, right = stack_left.pop(), stack_right.pop()
            if ((left is not None) ^ (right is not None)) or (left and left.val != right.val):
                return False
            if left:
                stack_left.extend([left.left, left.right])
                stack_right.extend([right.right, right.left])
        return True if not stack_left and not stack_right else False
```
