# 题目描述

[剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

为了让您更好地理解问题，以下面的二叉搜索树为例：

![](/Algorithm/imgs/bstdlloriginalbst.png)

我们希望将这个二叉搜索树转化为双向循环链表。链表中的每个节点都有一个前驱和后继指针。对于双向循环链表，第一个节点的前驱是最后一个节点，最后一个节点的后继是第一个节点。

下图展示了上面的二叉搜索树转化成的链表。“head” 表示指向链表中有最小元素的节点。

![](/Algorithm/imgs/bstdllreturndll.png)

特别地，我们希望可以就地完成转换操作。当转化完成以后，树中节点的左指针需要指向前驱，树中节点的右指针需要指向后继。还需要返回链表中的第一个节点的指针。

# 解题思路

二叉搜索树的中序遍历得到的就是一个递增的数组, 树中最小的数字排在列表的第一个, 因此也是第一个被找到的, 最大的数字排在列表的最后一个, 最后一个被找到, 加入到结果列表中.

因此这里我们用**中序遍历的递归形式**, 能够按大小顺序**依次定位**结点, 这正是问题所需要的, 找到了前后结点, 我们接下来要做的事情, 就是将前结点的`right`指针指向后结点, 后结点的`left`指针指向前结点, 组成双向链表. 最后再在头结点和尾结点之间建立双向的联系, 就完成了完整的双向链表的改造.

具体解释参考: [面试题36. 二叉搜索树与双向链表（中序遍历，清晰图解）](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/solution/mian-shi-ti-36-er-cha-sou-suo-shu-yu-shuang-xian-5/).

```python
"""
# Definition for a Node.
class Node:
    def __init__(self, val, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
"""

class Solution:
    def treeToDoublyList(self, root: 'Node') -> 'Node':
        def dfs(node):
            if not node:
                return

            dfs(node.left)
            if self.pre is None:
                self.head = node
            else:
                self.pre.right = node
                node.left = self.pre
            self.pre = node

            dfs(node.right)

        self.head, self.pre = None, None
        dfs(root)
        if self.head is not None:
            self.head.left = self.pre
            self.pre.right = self.head
        return self.head
```
