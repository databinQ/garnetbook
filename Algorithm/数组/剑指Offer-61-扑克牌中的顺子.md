# 题目描述

[剑指 Offer 61. 扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。

示例 1:
```
输入: [1,2,3,4,5]
输出: True
```

示例 2:
```
输入: [0,0,1,2,5]
输出: True
```

限制：

- 数组长度为 5
- 数组的数取值为 [0, 13]

# 解题思路

[面试题61. 扑克牌中的顺子（集合 Set / 排序，清晰图解）](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/solution/mian-shi-ti-61-bu-ke-pai-zhong-de-shun-zi-ji-he-se/)

```python
class Solution:
    def isStraight(self, nums: List[int]) -> bool:
        max_card, min_card = float('-inf'), float('inf')
        seen = set()
        for num in nums:
            if num == 0:
                continue
            if num in seen:
                return False
            seen.add(num)
            min_card = min(min_card, num)
            max_card = max(max_card, num)
        return max_card - min_card < 5
```
