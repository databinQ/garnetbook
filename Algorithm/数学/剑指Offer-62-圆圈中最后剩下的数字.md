# 题目描述

[剑指 Offer 62. 圆圈中最后剩下的数字](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)

0,1,,n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字。求出这个圆圈里剩下的最后一个数字。

例如，0、1、2、3、4这5个数字组成一个圆圈，从数字0开始每次删除第3个数字，则删除的前4个数字依次是2、0、4、1，因此最后剩下的数字是3。

示例 1：
```
输入: n = 5, m = 3
输出: 3
```

示例 2：
```
输入: n = 10, m = 17
输出: 2
```

限制：

- 1 <= n <= 10^5
- 1 <= m <= 10^6

# 解题思路

[Java解决约瑟夫环问题，告诉你为什么模拟会超时！](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/solution/javajie-jue-yue-se-fu-huan-wen-ti-gao-su-ni-wei-sh/)

## 迭代

生成一个长度为$$n$$的列表, 依次寻找当前应当被删除的数字, 将这个数字从列表中删除, 继续寻找下一个被删除的数字.

假设当前删除的位置是 $$idx$$，下一个删除的数字的位置是 $$idx + m$$ 。但是，由于把当前位置的数字删除了，后面的数字会前移一位，所以实际的下一个位置是 $$idx + m - 1$$。由于数到末尾会从头继续数，所以最后取模一下，就是 $$(idx + m - 1) \pmod n$$。

```python
class Solution:
    def lastRemaining(self, n: int, m: int) -> int:
        nums = list(range(n))
        index = 0
        while len(nums) > 1:
            index = (m + index - 1) % len(nums)
            nums.pop(index)
        return nums[0]
```

## 约瑟夫环

另一种解法是约瑟夫环, 具体参考上面链接中数学解法的部分.
