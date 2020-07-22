# 题目描述

[剑指 Offer 11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

剑指 Offer 11. 旋转数组的最小数字
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  

示例 1：
```
输入：[3,4,5,1,2]
输出：1
```

示例 2：
```
输入：[2,2,2,0,1]
输出：0
```

# 解题思路

## 二分法

[面试题11. 旋转数组的最小数字（二分法，清晰图解）](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/solution/mian-shi-ti-11-xuan-zhuan-shu-zu-de-zui-xiao-shu-3/)

```python
class Solution:
    def minArray(self, numbers: List[int]) -> int:
        n = len(numbers)
        left, right = 0, n - 1

        while left < right:
            mid = (left + right) // 2
            if numbers[mid] < numbers[right]:
                right = mid
            elif numbers[mid] > numbers[right]:
                left = mid + 1
            else:  # numbers[mid] == numbers[right]
                right -= 1
        return numbers[left]
```

# 相同题目

[[154][简单][二分] 寻找旋转排序数组中的最小值 II](/Algorithm/二分/154-寻找旋转排序数组中的最小值-II.md)
