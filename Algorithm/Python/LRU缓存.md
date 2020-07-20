# LRU Cache

LRU即**Least Recently Used**, 近期最少使用, 如名字, 作为一种缓存, 在容量受限的情况下, 会优先淘汰最少使用的对象.

在Python中, 位置在`functools.lru_cache`, 是一个**修饰器**, 用户对需要缓存结果的函数在定义时, 进行修饰. 在运行中, 作为一个代替`redis`的轻缓存, 以函数的入参为`key`, 以该入参运行得到的结果为`value`, 进行缓存. 调用此函数时, 如果入参在缓存中, 则直接返回结果, 不再调用函数, 大大加速了运行效率.

根据以上的特点, 可以得到, 想使用`lru_cache`, 必须满足:

- 入参必须是**不可变参数**, 如`int`, `str`, `tuple`等, `list`, `dict`等是无法使用的
   - 遇到这种情况, 只能通过闭包的方法解决
- 函数的映射关系是固定的, 即对于同一个输入, 输出必须是固定不变的. 对于一些有状态的函数可能无法使用

# 原理

`functools.lru_cache`是通过**一个双向链表**外加**一个字典**实现缓存的.

- 字典的作用是在$$O(1)$$的时间内找到结果
- 双向链表的作用是在$$O(1)$$的时间内删除掉最少使用的key

## 源码

### lru_cache

```python
def lru_cache(maxsize=128, typed=False):
  if maxsize is not None and not isinstance(maxsize, int):
  raise TypeError('Expected maxsize to be an integer or None')

  def decorating_function(user_function):
    wrapper = _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo)
    return update_wrapper(wrapper, user_function)

  return decorating_function
```

`lru_cache`接受两个参数:

- `maxsize`: 缓存的数量, 即双向链表的最大长度. 可以为`None`, 表示没有限制; 可以为`0`, 表示不使用缓存
- `typed`: 以入参作为缓存, 此参数是指是否对入参的类型做区分. 例如`3.0`和`3`, 在`typed=False`的情况下, 共用同一缓存

### lru_cache_wrapper

函数的逻辑是在`_lru_cache_wrapper`中实现的, 代码分段如下:

```python
def _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo):
    # Constants shared by all lru cache instances:
    sentinel = object()          # 特殊标记, 用来表示缓存未命中
    make_key = _make_key         # 根据入参来生成key的函数
    PREV, NEXT, KEY, RESULT = 0, 1, 2, 3   # 表示单个节点的4个字段

    cache = {}
    hits = misses = 0
    full = False
    cache_get = cache.get    # bound method to lookup a key or return None
    cache_len = cache.__len__  # get cache size without calling len()
    lock = RLock()           # because linkedlist updates aren't threadsafe
    root = []                # root of the circular doubly linked list
    root[:] = [root, root, None, None]     # initialize by pointing to self
```

`sentinel`是一个唯一的对象, 在从字典中获取缓存结果时, 如果对应的key不在字典中, 默认返回这个值. 之所以用这个值, 是因为`None`也是函数可能的返回值, 不能作为判定key是否在字典中的条件.

对于一个结点, 我们使用一个长度为4, 表示为`[prev, key, result, next]`的列表来存储, 每个位置代表的意义是固定的, 分别为双向链表上一个节点(实际上是一个列表), `key`值, 对应的结果值, 下一个结点. 因此用`PREV, NEXT, KEY, RESULT = 0, 1, 2, 3`常量来表示和记录每个位置, 及其意思.

`cache`就是记录数据的字典, 根据`key`获取结果.

`root`代表双向链表的根节点. 双向链表即**环形链表**, 往往使用一个**空节点**(没有key和value, 只连接前后结点)作为定位, 因此长度为$$N$$的双向链表, 其实只存储了$$N-1$$个数据点. 在后面操作更新状态时, 要维护好`root`节点.

#### maxsize == 0

```python
if maxsize == 0:
    def wrapper(*args, **kwds):
        # No caching -- just a statistics update after a successful call
        nonlocal misses
        result = user_function(*args, **kwds)
        misses += 1
        return result
```

这种情况就是不使用缓存, 直接调用函数返回即可.

#### maxsize is None

```python
elif maxsize is None:
    def wrapper(*args, **kwds):
        # Simple caching without ordering or size limit
        nonlocal hits, misses
        key = make_key(args, kwds, typed)
        result = cache_get(key, sentinel)
        if result is not sentinel:
            hits += 1
            return result
        result = user_function(*args, **kwds)
        cache[key] = result
        misses += 1
        return result
```

这种情况代表缓存大小没有限制, 在这种情况下就不涉及不常使用的key被淘汰的问题, 无需(也不能)使用双向链表, 直接从字典中获取缓存的值即可.

#### 有大小限制的缓存

```python
def wrapper(*args, **kwds):
    # Size limited caching that tracks accesses by recency
    nonlocal root, hits, misses, full
    key = make_key(args, kwds, typed)
    with lock:
        link = cache_get(key)
        if link is not None:
            # Move the link to the front of the circular queue
            link_prev, link_next, _key, result = link
            link_prev[NEXT] = link_next
            link_next[PREV] = link_prev
            last = root[PREV]
            last[NEXT] = root[PREV] = link
            link[PREV] = last
            link[NEXT] = root
            hits += 1
            return result
    result = user_function(*args, **kwds)
    with lock:
        if key in cache:
            # Getting here means that this same key was added to the
            # cache while the lock was released.  Since the link
            # update is already done, we need only return the
            # computed result and update the count of misses.
            pass
        elif full:
            # Use the old root to store the new key and result.
            oldroot = root
            oldroot[KEY] = key
            oldroot[RESULT] = result
            # Empty the oldest link and make it the new root.
            # Keep a reference to the old key and old result to
            # prevent their ref counts from going to zero during the
            # update. That will prevent potentially arbitrary object
            # clean-up code (i.e. __del__) from running while we're
            # still adjusting the links.
            root = oldroot[NEXT]
            oldkey = root[KEY]
            oldresult = root[RESULT]
            root[KEY] = root[RESULT] = None
            # Now update the cache dictionary.
            del cache[oldkey]
            # Save the potentially reentrant cache[key] assignment
            # for last, after the root and links have been put in
            # a consistent state.
            cache[key] = oldroot
        else:
            # Put result in a new link at the front of the queue.
            last = root[PREV]
            link = [last, root, key, result]
            last[NEXT] = root[PREV] = cache[key] = link
            # Use the cache_len bound method instead of the len() function
            # which could potentially be wrapped in an lru_cache itself.
            full = (cache_len() >= maxsize)
        misses += 1
    return result
```

这种情况, 才是正常的情况. 上面是完整的代码, 拆开来看.

##### 获取缓存

```python
with lock:
    link = cache_get(key)
    if link is not None:
        # Move the link to the front of the circular queue
        link_prev, link_next, _key, result = link
        link_prev[NEXT] = link_next
        link_next[PREV] = link_prev
        last = root[PREV]
        last[NEXT] = root[PREV] = link
        link[PREV] = last
        link[NEXT] = root
        hits += 1
        return result
```

根据入参得到对应的`key`, 从字段中尝试获取缓存的结果. 如果是有对应缓存的, 在返回结果之前, 要更新双向链表. 这是因为我们激活了`key`值, 需要将这个`key`移动到代表最新的位置. 代码中对应的是`root`前面(`prev`)的结点.

- `link_prev, link_next, _key, result = link`: 得到`key`对应结点的前后结点`link`
- `link_prev[NEXT] = link_next`, `link_next[PREV] = link_prev`: 将`key`的前后结点相接, 因为`key`结点移走了
- `last = root[PREV]`: `root`之前的结点, 是刚才最新的结点, 获取这个结点, 接下来要把`key`这个结点插入到`last`和`root`之间
- `last[NEXT] = root[PREV] = link`: `root`之前的结点更新为`link`, `last`之后的结点更新为`link`, 完成了`link`之外相关结点的更新, 接下来更新`link`本身
- `link[PREV] = last`, `link[NEXT] = root`: 更新`link`本身, 双向链表调整完成

##### 执行函数

```python
result = user_function(*args, **kwds)
```

只有在缓存中不存在时才会执行.

##### 写入缓存

执行完函数, 就要把函数运行的结果和对应的`key`写入到缓存中.

```python
if key in cache:
    # Getting here means that this same key was added to the
    # cache while the lock was released.  Since the link
    # update is already done, we need only return the
    # computed result and update the count of misses.
    pass
```

这里考虑了多线程的情况. 可能当前线程执行和另一个线程执行同一入参, 之前缓存不存在, 两个线程都去执行了函数. 但另一个线程快一些, 已经执行完毕, 并且写入了缓存. 轮到此线程时, 再判断一下是否已经写入到缓存了, 如果已经写入, 就不用再费心思, 跳过.

##### 队列已满

```python
elif full:
    # Use the old root to store the new key and result.
    oldroot = root
    oldroot[KEY] = key
    oldroot[RESULT] = result
    # Empty the oldest link and make it the new root.
    # Keep a reference to the old key and old result to
    # prevent their ref counts from going to zero during the
    # update. That will prevent potentially arbitrary object
    # clean-up code (i.e. __del__) from running while we're
    # still adjusting the links.
    root = oldroot[NEXT]
    oldkey = root[KEY]
    oldresult = root[RESULT]
    root[KEY] = root[RESULT] = None
    # Now update the cache dictionary.
    del cache[oldkey]
    # Save the potentially reentrant cache[key] assignment
    # for last, after the root and links have been put in
    # a consistent state.
    cache[key] = oldroot
```

写入新缓存时队列已满, 我们就要剔除掉最旧的`key`. 环形链表中, 所有的`key`按操作时间排序, 以`root`为界限, `root`之前是最新的结点, 那么`root`之后是最旧的结点, 应该被剔除掉. 剔除的方法, 是把`root`结点转移到最旧的结点(即`root[NETX]`)上, 新的`key`对应的结点覆盖原来`root`的位置.

- `oldroot = root`: 获取此时的根节点, 即旧的`root`节点, `oldroot`
- `oldroot[KEY] = key`, `oldroot[RESULT] = result`: 将新的`key`的信息保存到`oldroot`上. 这是因为`root`节点都是空节点, 没有什么好转移的, 直接覆盖就好
- `root = oldroot[NEXT]`: 新的`root`结点选取为原来的根节点`oldroot`之后的结点
- `oldkey = root[KEY]`, `oldresult = root[RESULT]`: 获取要删除的, 最旧结点的`key`值, 后面在缓存字典`cache`中删掉
- `root[KEY] = root[RESULT] = None`: `root`结点没有`key`, `result`, 清空
- `del cache[oldkey]`: 从缓存中删除最旧的`key`
- `cache[key] = oldroot`: 缓存字典中装入最新的`key`

##### 队列未满

可能是有限长度缓存还未装满, 或者缓存的大小就没有限制. 只需要在`root`和`root[PREV]`之间插入一个新的结点即可. 最后更新一下链表是否已满的状态.

```python
else:
    # Put result in a new link at the front of the queue.
    last = root[PREV]
    link = [last, root, key, result]
    last[NEXT] = root[PREV] = cache[key] = link
    # Use the cache_len bound method instead of the len() function
    # which could potentially be wrapped in an lru_cache itself.
    full = (cache_len() >= maxsize)
```

# 参考资料

- [Python 实现LRU Cache](https://www.cnblogs.com/dion-90/articles/8540787.html)
- [python 刷题小技巧之 lru_cache](https://zhuanlan.zhihu.com/p/79544192)
