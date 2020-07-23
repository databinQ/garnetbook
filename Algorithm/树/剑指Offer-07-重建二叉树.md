# 题目描述

[剑指 Offer 07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

例如，给出
```
前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
```

返回如下的二叉树：
```
    3
   / \
  9  20
    /  \
   15   7
```

限制：

- 0 <= 节点个数 <= 5000

# 解题思路

首先找到根节点, 然后分别确定左子树对应的遍历数组, 以及右子树的遍历数组. 单独观察左右子树, 其遍历方法同完整的树, 因此可以递归地分解拼接. 找到根节点, 确定左右子树数组, 得到左右子树, 拼接到根节点左右. 对每层树都这么进行操作.

找到根节点. 前序遍历数组中的第一个数字就是根节点.

然后确定左右子树对应的列表. 中序遍历中, 根节点所在的位置, 将左右子树中的节点分隔开来, 因此找到根节点的值在中序遍历数组中的位置, 这个位置之前的数组属于左子树, 之后属于右子树. 这样得到了中序遍历的左右子树列表.

接下来需要确定前序遍历中左右子树的列表. 通过上一步, 得到了左子树中节点的数量(即左子树列表的长度), 而前序遍历数组是按**根节点+左子树节点+右子树节点**的顺序构成的, 因此按长度就能分割出左右子树对应的前序遍历数组的长度.

递归地构造子树, 拼接在根节点左右.

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> TreeNode:
        if not preorder:
            return None

        root_value = preorder[0]
        t_index = inorder.index(root_value)
        left_length = len(inorder[:t_index])

        root = TreeNode(root_value)
        root.left = self.buildTree(preorder[1: 1 + left_length], inorder[:t_index])
        root.right = self.buildTree(preorder[1 + left_length:], inorder[t_index + 1:])
        return root
```
