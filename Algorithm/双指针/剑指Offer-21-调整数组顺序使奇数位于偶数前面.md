# 题目描述

[剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

示例：

输入：nums = [1,2,3,4]
输出：[1,3,2,4]
注：[3,1,2,4] 也是正确的答案之一。

提示：

- 1 <= nums.length <= 50000
- 1 <= nums[i] <= 10000

# 解题思路

## 双指针

### 快慢指针

```python
class Solution:
    def exchange(self, nums: List[int]) -> List[int]:
        n = len(nums)
        odd, even = 0, 0
        while odd < n:
            if nums[odd] % 2 == 1:
                nums[odd], nums[even] = nums[even], nums[odd]
                even += 1
            odd += 1
        return nums
```

### 首尾指针

[面试题21. 调整数组顺序使奇数位于偶数前面（双指针，清晰图解）](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/solution/mian-shi-ti-21-diao-zheng-shu-zu-shun-xu-shi-qi-4/)

```python
class Solution:
    def exchange(self, nums: List[int]) -> List[int]:
        i, j = 0, len(nums) - 1
        while i < j:
            while i < j and nums[i] & 1 == 1: i += 1
            while i < j and nums[j] & 1 == 0: j -= 1
            nums[i], nums[j] = nums[j], nums[i]
        return nums
```
