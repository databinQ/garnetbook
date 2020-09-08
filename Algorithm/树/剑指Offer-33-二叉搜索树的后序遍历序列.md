# 题目描述

[剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 true，否则返回 false。假设输入的数组的任意两个数字都互不相同。

参考以下这颗二叉搜索树：
```
     5
    / \
   2   6
  / \
 1   3
```

示例 1：
```
输入: [1,6,3,2,5]
输出: false
```

示例 2：
```
输入: [1,3,2,6,5]
输出: true
```

# 解题思路

## 分治

我们把当前的数组作为争取的后续遍历产生的, 那么数组的最后一个位置就是整棵树的根节点, 在从数组的首位向后遍历, 找到第一个大于根节点的数(题目限定数组中任何两个数不等), 则这个数之前是左子树, 包括这个数在内, 右面是右子树, 这就完成了分.

由于是从左向右遍历, 此时左侧的数值肯定都是小与根节点值的, 要判断的是右侧子数组中的每个数组是否都大于根节点的值, 只要有小于的, 就不符合搜索树后序遍历的结果. 因此只需要遍历右子树, 判断是否符合条件.

然后递归地判断左右子树是否符合条件.

下面的代码是将左右子数组(子树)一视同仁, 进行分治递归的思路.

```python
class Solution:
    def verifyPostorder(self, postorder: List[int]) -> bool:
        def dfs(nodes):
            n = len(nodes)
            if n == 0:
                return True, None, None
            if n == 1:
                return True, nodes[0], nodes[0]

            root = nodes[-1]
            first = 0
            for first in range(n):
                if nodes[first] > root:
                    break
            left = nodes[:first]
            right = nodes[first: n - 1]

            lres, lmax, lmin = dfs(left)
            if not lres or (lmax is not None and lmax > root):
                return False, None, None
            rres, rmax, rmin = dfs(right)
            if not rres or (rmin is not None and rmin < root):
                return False, None, None
            return True, rmax if rmax is not None else root, lmin if lmin is not None else root

        return dfs(postorder)[0]
```
