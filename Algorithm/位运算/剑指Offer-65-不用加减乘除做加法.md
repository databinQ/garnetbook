# 题目描述

[剑指 Offer 65. 不用加减乘除做加法](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

写一个函数，求两个整数之和，要求在函数体内不得使用 “+”、“-”、“*”、“/” 四则运算符号。

示例:
```
输入: a = 1, b = 1
输出: 2
```

提示：

- a, b 均可能是负数或 0
- 结果不会溢出 32 位整数

# 解题思路

对于两个数字`a`和`b`, 有:

- `a^b`: 每位的异或运算, 得到的结果为**无进位相加的结果**
- `a&b`: 每位的与运算, 得到的结果为**进位结果**

因此将这`a^b`与`a&b`**左移一位**相加, 得到的仍然是`a+b`. 左移一位的目的是实现进位.

按照$$s = a + b \Rightarrow s = n + c$$这个思路, 循环求$$n$$和$$c$$, 当进位$$c$$为0时, 对应的$$n$$就是最终的加和结果.

Python中对于负数位的处理参考: [面试题65. 不用加减乘除做加法（位运算，清晰图解）](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/solution/mian-shi-ti-65-bu-yong-jia-jian-cheng-chu-zuo-ji-7/).

```python
class Solution:
    def add(self, a: int, b: int) -> int:
        x = 0xffffffff
        a, b = a & x, b & x
        while b != 0:
            a, b = a ^ b, (a & b) << 1 & x
        return a if a <= 0x7fffffff else ~(a ^ x)
```
