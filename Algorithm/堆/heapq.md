# heapq

**heapq**包提供了高效的堆算法(也称为**优先队列算法**).

**堆**, 即一种二叉树结构, 其中每个父结点的值都小于或等于它的两个子结点, 两个子结点之间大小没有要求. 使用数组存储堆结构时, 对于第$$i$$个结点, 其左右子结点的索引分别为$$2i+1$$与$$2i+2$$.

堆分为最大堆和最小堆, heapq提供的是**固定的最小堆**, 即堆顶元素`heap[0]`为整个堆中的最小值.

## 使用方法

### 创建堆

`heapq`使用python中的`list`数据结构来构建堆. 有两种创建堆的方式:

- 初始化一个空的`list`: `[]`, `heapq`提供的方法, 如`push`, `pop`, `replace`等方法, 都需要传入一个`list`, 整个`list`作为堆来使用
- 使用`heapify(list)`方法, 将现有的一个list整理成堆

#### 从空堆开始创建

```python
>>> from heapq import *
>>> def heapsort(iterable):
...     h = []
...     for value in iterable:
...         heappush(h, value)
...     return [heappop(h) for i in range(len(h))]
...
>>> heapsort([1, 3, 5, 7, 9, 2, 4, 6, 8, 0])
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

除了放入纯数字, 可以带入一些业务信息

```python
>>> h = []
>>> heappush(h, (5, 'write code'))
>>> heappush(h, (7, 'release product'))
>>> heappush(h, (1, 'write spec'))
>>> heappush(h, (3, 'create tests'))
>>> heappop(h)
(1, 'write spec')
```

### 常用方法

参考文档[8.4. heapq — Heap queue algorithm](https://docs.python.org/2/library/heapq.html).
