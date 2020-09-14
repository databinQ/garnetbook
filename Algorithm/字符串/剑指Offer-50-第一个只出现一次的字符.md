# 题目描述

[剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。

示例:
```
s = "abaccdeff"
返回 "b"

s = ""
返回 " "
```

限制：

- 0 <= s 的长度 <= 50000

# 解题思路

Python3中的字段是有序的, 只需要遍历字典即可, 不需要遍历字符串, 挨个字符判断是否符合.

```python
class Solution:
    def firstUniqChar(self, s: str) -> str:
        cache = dict()
        for c in s:
            cache[c] = cache.get(c, 0) + 1
        for k, v in cache.items():
            if v == 1:
                return k
        return ' '
```
