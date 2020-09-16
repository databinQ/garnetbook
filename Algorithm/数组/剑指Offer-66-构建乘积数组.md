# 题目描述

[剑指 Offer 66. 构建乘积数组]https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/

给定一个数组 A[0,1,…,n-1]，请构建一个数组 B[0,1,…,n-1]，其中 B 中的元素 B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]。不能使用除法。

示例:
```
输入: [1,2,3,4,5]
输出: [120,60,40,30,24]
```

提示：

- 所有元素乘积之和不会溢出 32 位整数
- a.length <= 100000

# 解题思路

[面试题66. 构建乘积数组（表格分区，清晰图解）](https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/solution/mian-shi-ti-66-gou-jian-cheng-ji-shu-zu-biao-ge-fe/)

本题的关键在于**不能使用除法**, 因此使用类似于滑动窗口的方式, 除以当前值, 乘上上一个数字的方法不能使用.

在只能使用乘法的情况下, 详细的解题方法参考上面的连接.

```python
class Solution:
    def constructArr(self, a: List[int]) -> List[int]:
        temp, n = 1, len(a)        
        b = [1] * n
        for i in range(1, n):  # 计算下三角
            b[i] = b[i - 1] * a[i - 1]
        for i in range(n - 2, -1, -1):
            temp *= a[i + 1]
            b[i] = b[i] * temp
        return b
```
