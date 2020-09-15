# 题目描述

[剑指 Offer 59 - II. 队列的最大值](https://leetcode-cn.com/problems/dui-lie-de-zui-da-zhi-lcof/)

请定义一个队列并实现函数 max_value 得到队列里的最大值，要求函数max_value、push_back 和 pop_front 的均摊时间复杂度都是O(1)。

若队列为空，pop_front 和 max_value 需要返回 -1

示例 1：
```
输入:
["MaxQueue","push_back","push_back","max_value","pop_front","max_value"]
[[],[1],[2],[],[],[]]
输出: [null,null,null,2,1,2]
```

示例 2：
```
输入:
["MaxQueue","pop_front","max_value"]
[[],[],[]]
输出: [null,-1,-1]
```

限制：

- 1 <= push_back,pop_front,max_value的总操作数 <= 10000
- 1 <= value <= 10^5

# 解题思路

使用一个辅组队列, 保存队列的头部是当前真正队列中的最大值. 具体方法参考[[239][困难][队列] 滑动窗口最大值](/Algorithm/队列/239-滑动窗口最大值.md).

```python
class MaxQueue:

    def __init__(self):
        self.queue = deque()
        self.max_queue = []

    def max_value(self) -> int:
        return self.max_queue[0] if self.max_queue else -1

    def push_back(self, value: int) -> None:
        self.queue.append(value)
        while self.max_queue and self.max_queue[-1] < value:
            self.max_queue.pop()
        self.max_queue.append(value)

    def pop_front(self) -> int:
        if not self.queue:
            return -1
        value = self.queue.popleft()
        if self.max_queue and self.max_queue[0] == value:
            self.max_queue.pop(0)
        return value


# Your MaxQueue object will be instantiated and called as such:
# obj = MaxQueue()
# param_1 = obj.max_value()
# obj.push_back(value)
# param_3 = obj.pop_front()
```
