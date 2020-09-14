# 题目描述

[剑指 Offer 57 - II. 和为s的连续正数序列](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)

输入一个正整数 target ，输出所有和为 target 的连续正整数序列（至少含有两个数）。

序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

示例 1：
```
输入：target = 9
输出：[[2,3,4],[4,5]]
```

示例 2：
```
输入：target = 15
输出：[[1,2,3,4,5],[4,5,6],[7,8]]
```

限制：

- 1 <= target <= 10^5

# 解题思路

## 双指针

我们定义两个指针, 控制窗口, 计算窗口中所有数字之和, 等于`target`的区间, 加入到结果中去. 由于题目要求区间至少含有两个数, 我们从`1`, `2`两个值开始 ,设立两个指针. 区间停止条件是右指针越过`target // 2 + 1`, 这是因为区间至少有两个数, 大于这个边界的两个值肯定是大于`target`的, 继续遍历只会更大, 也无需继续遍历.

区间数字之和使用等差数列公式计算. 遍历过程中遇到和值等于`target`的, 将区间添加到最终的结果列表中; 和值小于`target`的, 向右移动右指针; 和值大于`target`的, 向右移动左指针. 中间过程我们不会遇到单个值区间符合`target`的情况, 因为这些数都小于`target`, 不用考虑这里的边际情况.

```python
class Solution:
    def findContinuousSequence(self, target: int) -> List[List[int]]:
        res = []
        left, right = 1, 2
        while right <= target // 2 + 1:
            sum_res = (left + right) / 2 * (right - left + 1)
            if sum_res == target:
                res.append(list(range(left, right + 1)))
                right += 1
            elif sum_res < target:
                right += 1
            else:
                left += 1
        return res
```
