# 题目描述

[剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/)

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

示例 1:
```
输入: [0,1,3]
输出: 2
```

示例 2:
```
输入: [0,1,2,3,4,5,6,7,9]
输出: 8
```

限制：

- 1 <= 数组长度 <= 10000

# 解题思路

递增数组用二分. 如果不缺失数字, 数组中索引对应的数字就会等于索引, 所以我们找到**第一个值与索引不匹配的位置**, 就是缺失的数字.

二分过程中, 遇到值与索引相等的, 移动左指针, 否则移动右指针.

```python
class Solution:
    def missingNumber(self, nums: List[int]) -> int:
        left, right = 0, len(nums) - 1
        while left <= right:
            mid = (left + right) // 2
            if nums[mid] > mid:
                right = mid - 1
            elif nums[mid] == mid:
                left = mid + 1
        return left
```
