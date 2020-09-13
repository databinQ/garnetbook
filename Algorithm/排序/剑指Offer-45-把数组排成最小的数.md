# 题目描述

[剑指 Offer 45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

输入一个非负整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

示例 1:
```
输入: [10,2]
输出: "102"
```

示例 2:
```
输入: [3,30,34,5,9]
输出: "3033459"
```

提示:

- 0 < nums.length <= 100

说明:

- 输出结果可能非常大，所以你需要返回一个字符串而不是整数
- 拼接起来的数字可能会有前导 0，最后结果不需要去掉前导 0

# 解题思路

[面试题45. 把数组排成最小的数（自定义排序，清晰图解）](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/solution/mian-shi-ti-45-ba-shu-zu-pai-cheng-zui-xiao-de-s-4/)

```python
class Solution:
    def minNumber(self, nums: List[int]) -> str:
        def compare(num1, num2):
            a, b = num1 + num2, num2 + num1
            if a > b:
                return 1
            elif a < b:
                return -1
            else:
                return 0
            for i in range(min(n1, n2)):
                if num1[i] > num2[i]:
                    return 1
                elif num1[i] < num2[i]:
                    return -1
            return 1 if n1 < n2 else 0 if n1 == n2 else -1
        max_count = max([len(str(num)) for num in nums])
        new_nums = sorted([str(num) for num in nums], key=functools.cmp_to_key(compare))
        return ''.join(new_nums)
```
