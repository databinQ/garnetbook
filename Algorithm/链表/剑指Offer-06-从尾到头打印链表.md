# 题目描述

[剑指 Offer 06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

示例 1：
```
输入：head = [1,3,2]
输出：[2,3,1]
```

限制：

- 0 <= 链表长度 <= 10000

# 解题思路

反转链表使用**递归**或**栈**.

递归的解法:

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def reversePrint(self, head: ListNode) -> List[int]:
        return self.reversePrint(head.next) + [head.val] if head is not None else []
```
