# 题目描述

[剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)

B是A的子结构， 即 A中有出现和B相同的结构和节点值。

例如:
```
给定的树 A:

     3
    / \
   4   5
  / \
 1   2
给定的树 B：

   4 
  /
 1
返回 true，因为 B 与 A 的一个子树拥有相同的结构和节点值。
```

示例 1：
```
输入：A = [1,2,3], B = [3,1]
输出：false
```

示例 2：
```
输入：A = [3,4,5,1,2], B = [4,1]
输出：true
```

限制：

- 0 <= 节点个数 <= 10000

# 解题思路

两个递归函数.

第一个递归函数是外层递归, 下面代码中的`dfs`, 判断一个树是否是另一个树的子树. 终止条件为:

- 当 树 A 为空 或 树 B 为空 时，直接返回 false

递归逻辑为:

- 两个树的根节点的值不相同时, 判断B是否在A的左子树或右子树中
- 两个树的根节点值相等, 首先判断以A为根节点的子树, 是否包含树B
  - 包含则找到了结果, 返回
  - 没有包含, 则继续寻找, 判断B是否在A的左子树或右子树中

第二个递归函数是内层递归, 下面代码中的`equal`, 判断树A包含当前根节点的子树, 是否包含树B. 终止条件为:

- 当节点 B 为空：说明树 B 已匹配完成（越过叶子节点），因此返回 true
- 当节点 A 为空：说明已经越过树 A 叶子节点，即匹配失败，返回 false
- 当节点 A 和 B 的值不同：说明匹配失败，返回 false

递归逻辑为:

- 判断A的左子树和B的左子树是否相等(包含), 且A的右子树和B的右子树是否相等(包含)

[面试题26. 树的子结构（先序遍历 + 包含判断，清晰图解）](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/solution/mian-shi-ti-26-shu-de-zi-jie-gou-xian-xu-bian-li-p/)

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def isSubStructure(self, A: TreeNode, B: TreeNode) -> bool:
        def equal(tree1, tree2):
            if tree2 is None:
                return True
            if tree1 is None:
                return False
            if tree1.val != tree2.val:
                return False
            return equal(tree1.left, tree2.left) and equal(tree1.right, tree2.right)

        def dfs(tree1, tree2):
            if tree1 is None or tree2 is None:
                return False

            if tree1.val != tree2.val:
                return dfs(tree1.left, tree2) or dfs(tree1.right, tree2)

            if equal(tree1, tree2):
                return True
            else:
                return dfs(tree1.left, tree2) or dfs(tree1.right, tree2)
        return dfs(A, B)
```
