# 题目描述

[剑指 Offer 60. n个骰子的点数](https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/)


把n个骰子扔在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。

你需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。

示例 1:
```
输入: 1
输出: [0.16667,0.16667,0.16667,0.16667,0.16667,0.16667]
```

示例 2:
```
输入: 2
输出: [0.02778,0.05556,0.08333,0.11111,0.13889,0.16667,0.13889,0.11111,0.08333,0.05556,0.02778]
```

限制：

- 1 <= n <= 11

# 解题思路

[动态规划（扫了一圈，俺是最短的）](https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/solution/java-dong-tai-gui-hua-by-zhi-xiong/)

对于$$n$$个骰子掷出的情况, 可以分解为$$n-1$$个骰子再加上单独一个骰子的表现情况. 这样就找到了状态转移的联系. 另外初始化概率分布数组为1个骰子的情况, 即`[0.16667,0.16667,0.16667,0.16667,0.16667,0.16667]`. 具体的状态转移关系见上面的连接.

一个规律是对于$$n$$个骰子, 可能的出现的和值的数量为$$5 \times n + 1$$.

```python
class Solution:
    def twoSum(self, n: int) -> List[float]:
        dp = [1 / 6] * 6
        for i in range(2, n + 1):
            probs = [.0] * (5 * i + 1)
            for j in range(len(dp)):
                for k in range(6):
                    probs[j + k] += dp[j] / 6
            dp = probs
        return dp
```
