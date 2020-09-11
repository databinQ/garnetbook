# 题目描述

[剑指 Offer 38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

输入一个字符串，打印出该字符串中字符的所有排列。

你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。

示例:
```
输入：s = "abc"
输出：["abc","acb","bac","bca","cab","cba"]
```

限制：

- 1 <= s 的长度 <= 8

# 解题思路

[面试题38. 字符串的排列（回溯法，清晰图解）](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/solution/mian-shi-ti-38-zi-fu-chuan-de-pai-lie-hui-su-fa-by/)

```python
class Solution:
    def permutation(self, s: str) -> List[str]:
        chars, n = list(s), len(s)
        if n == 0:
            return []

        res = []

        def dfs(index):
            if index == n - 1:  # 遍历到了最后一个字符
                res.append(''.join(chars))
                return

            seen = set()
            for i in range(index, n):
                if chars[i] in seen:
                    continue
                chars[index], chars[i] = chars[i], chars[index]
                seen.add(chars[index])
                dfs(index + 1)
                chars[index], chars[i] = chars[i], chars[index]

        dfs(0)
        return res
```
