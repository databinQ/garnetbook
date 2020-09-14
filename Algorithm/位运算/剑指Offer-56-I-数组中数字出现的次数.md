# 题目描述

[剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

一个整型数组 nums 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

示例 1：
```
输入：nums = [4,1,4,6]
输出：[1,6] 或 [6,1]
```

示例 2：
```
输入：nums = [1,2,10,4,1,4,3,3]
输出：[2,10] 或 [10,2]
```

限制：

- 2 <= nums.length <= 10000

# 解题思路

[Leetcode官方题解: 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/solution/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-by-leetcode/)

```python
class Solution:
    def singleNumbers(self, nums: List[int]) -> List[int]:
        res = functools.reduce(lambda x, y: x ^ y, nums)
        inter = 1
        while inter & res == 0:
            inter <<= 1

        a, b = 0, 0
        for num in nums:
            if num & inter == 0:
                a ^= num
            else:
                b ^= num
        return [a, b]
```
