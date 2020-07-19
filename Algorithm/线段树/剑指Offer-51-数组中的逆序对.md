# 题目描述

[剑指 Offer 51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。

示例 1:
```
输入: [7,5,6,4]
输出: 5
```

限制：

- 0 <= 数组长度 <= 50000

# 解题思路

[[315][困难][线段树][二分] 计算右侧小于当前元素的个数](/Algorithm/线段树/315-计算右侧小于当前元素的个数.md)是这道题的等价题, 只不过是要计算每个数字对应的**逆序对**数量. 把数组中所有数字的逆序对数量加起来就是这道题最后的结果了.

```python
class Solution:
    def reversePairs(self, nums: List[int]) -> int:
        A = list(set(nums))
        A.sort()
        m = len(A)
        C = [0] * m
        mapping = {n: i + 1 for i, n in enumerate(A)}
        res = []

        for i in range(len(nums) - 1, -1, -1):
            num = nums[i]
            c_index = mapping[num]
            while c_index <= m:
                C[c_index - 1] += 1
                c_index += c_index & (-c_index)

            t_res = 0
            c_index = mapping[num] - 1
            while c_index > 0:
                t_res += C[c_index - 1]
                c_index -= c_index & (-c_index)
            res.append(t_res)

        return sum(res)
```

# 相关题目

- [[315][困难][线段树][二分] 计算右侧小于当前元素的个数](/Algorithm/线段树/315-计算右侧小于当前元素的个数.md)
