# 题目描述

[剑指 Offer 46. 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)

给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。

示例 1:
```
输入: 12258
输出: 5
解释: 12258有5种不同的翻译，分别是"bccfi", "bwfi", "bczi", "mcfi"和"mzi"
```

提示：

- 0 <= num < 231

# 解题思路

## 动态规划

动态规划的定义比较明确, `dp[i]`表示的就是前`i`位的子串可以有多少中翻译方法. 对于`dp[i+1]`, 将`s[i+1]`与`s[:i+1]`是一种肯定可行的拼接思路, 另一种是将`s[i:i+2]`这两个字符与`s[:i]`拼接, 要求这两个字符组成的数字不能以`0`开头, 且大小小于26(`z`字符对应25), 这样就得到了状态转移公式.

最后直接返回`dp[n]`即可, `n`是数字的长度.

```python
class Solution:
    def translateNum(self, num: int) -> int:
        string = str(num)
        f1, f2 = 1, 1
        for i, char in enumerate(string[1:]):
            count = f1
            if string[i] != '0' and int(string[i: i + 2]) < 26:
                count += f2
            f1, f2 = count, f1
        return f1
```
