## Python 库 - functools lru_cache

缓存在实际开发工作中的应用是非常常见的，比如内存缓存、redis缓存、本地文件缓存等，其主要的目的是方便数据的下一次访问，减少应用程序重复计算的次数和资源消耗，缓存是一种典型的空间换时间的方案。那么我们就来看看在开发Python程序时，functools提供的 [cache](https://docs.python.org/zh-cn/3/library/functools.html) 装饰器是如何工作的。

### 从一个小例子开始

首先我们看一下阶乘的实现：

```
def factorial(n):
    print('compute: {}'.format(n))
    return n * factorial(n - 1) if n else 1


a = factorial(5)
print(a)
b = factorial(10)
print(b)

compute: 5
compute: 4
compute: 3
compute: 2
compute: 1
compute: 0
120
compute: 10
compute: 9
compute: 8
compute: 7
compute: 6
compute: 5
compute: 4
compute: 3
compute: 2
compute: 1
compute: 0
3628800
```

我们分别计算了 `5！` 和 `10！` ，从程序的运行输出看，在计算 `10!` 时，把 `5!` 重复计算了一遍，在计算阶乘数字较小的情况下是没问题的，但是当阶乘数非常大时，重复的计算会占用CPU很多资源，而这些重复的计算完全是没必要的，那我们会想到实现一个缓存，将计算过的数据存起来，当下次请求时直接返回。

### 实现一个缓存

```
class MyCache(object):

    def __init__(self):
        self.cache = {}

    def get_cache(self, key) -> object:
        return self.cache.get(key, None)

    def set_cache(self, key, value) -> bool:
        if key not in self.cache:
            self.cache[key] = value
        return True

    def is_in_cache(self, key: str) -> bool:
        return key in self.cache

    def get_all_cache(self) -> dict:
        return self.cache

def factorial(n: int, cache: MyCache):
    print('compute: {}'.format(n))
    if cache.is_in_cache(n):
        print('hit cache: {}'.format(n))
        return cache.get_cache(n)
    tmp = n * factorial(n - 1, cache) if n else 1
    cache.set_cache(n, tmp)
    return tmp

cache = MyCache()
a = factorial(5, cache)
print(a)
b = factorial(10, cache)
print(b)
c = factorial(4, cache)
print(c)
print(cache.get_all_cache())
```

我们简单实现了一个缓存类，使用了 `dict` 数据类型，存储了中间的计算结果，我们看下输出：

```
compute: 5
compute: 4
compute: 3
compute: 2
compute: 1
compute: 0
120
compute: 10
compute: 9
compute: 8
compute: 7
compute: 6
compute: 5
hit cache: 5
3628800
compute: 4
hit cache: 4
24
{0: 1, 1: 1, 2: 2, 3: 6, 4: 24, 5: 120, 6: 720, 7: 5040, 8: 40320, 9: 362880, 10: 3628800}
```

 从输出能看出，`5!` 和 `4!` 都命中了缓存，直接拿到了结果，这就节省了CPU的计算资源，当然，如果数据量很小，就没有必要使用缓存了。

### functools中的lru_cache

从3.2版本开始，Python的functools模块提供了 `lru_cache` 函数，用于缓存计算的中间结果，函数定义如下：

```
@functools.lru_cache(user_function)
@functools.lru_cache(maxsize=128, typed=False)
```

一个为函数提供缓存功能的装饰器，缓存 `maxsize` 传入参数，在下次以相同参数调用时直接返回上一次的结果。用以节约高开销或I/O函数的调用时间。由于使用了字典存储缓存，所以该函数的固定参数和关键字参数必须是可哈希的。

不同模式的参数可能被视为不同从而产生多个缓存项，例如, f(a=1, b=2) 和 f(b=2, a=1) 因其参数顺序不同，可能会被缓存两次。

如果指定了 `user_function`，它必须是一个可调用对象。 这允许 `lru_cache` 装饰器被直接应用于一个用户自定义函数，`maxsize` 默认值为128。

我们使用这个装饰器重写一下上边的代码：

```
from functools import lru_cache

@lru_cache(None)
def factorial(n: int):
    return n * factorial(n - 1) if n else 1


a = factorial(5)
print(a)
b = factorial(10)
print(b)
c = factorial(4)
print(c)

print(factorial.cache_info())

120
3628800
24
CacheInfo(hits=2, misses=11, maxsize=None, currsize=11)
```

从输出可以看到，`hits=2` 表示命中了2次缓存。

那我们就来看下这个装饰器的实现：

```
def lru_cache(maxsize=128, typed=False):
    """Least-recently-used cache decorator.

    If *maxsize* is set to None, the LRU features are disabled and the cache
    can grow without bound.

    If *typed* is True, arguments of different types will be cached separately.
    For example, f(3.0) and f(3) will be treated as distinct calls with
    distinct results.

    Arguments to the cached function must be hashable.

    View the cache statistics named tuple (hits, misses, maxsize, currsize)
    with f.cache_info().  Clear the cache and statistics with f.cache_clear().
    Access the underlying function with f.__wrapped__.

    See:  http://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)

    """

    # Users should only access the lru_cache through its public API:
    #       cache_info, cache_clear, and f.__wrapped__
    # The internals of the lru_cache are encapsulated for thread safety and
    # to allow the implementation to change (including a possible C version).

    # Early detection of an erroneous call to @lru_cache without any arguments
    # resulting in the inner function being passed to maxsize instead of an
    # integer or None.  Negative maxsize is treated as 0.
    if isinstance(maxsize, int):
        if maxsize < 0:
            maxsize = 0
    elif maxsize is not None:
        raise TypeError('Expected maxsize to be an integer or None')

    def decorating_function(user_function):
        wrapper = _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo)
        return update_wrapper(wrapper, user_function)

    return decorating_function
```

如果 `maxsize` 设为 `None`，LRU 特性将被禁用且缓存可无限增长。

如果 `typed` 设置为true，不同类型的函数参数将被分别缓存。例如， `f(3)` 和 `f(3.0)` 将被视为不同而分别缓存。

被装饰的函数带有一个 `cache_info()` 函数。当调用 `cache_info()` 函数时，返回一个具名元组，包含命中次数 `hits`，未命中次数 `misses` ，最大缓存数量 `maxsize` 和 当前缓存大小 `currsize`。在多线程环境中，命中数与未命中数是不完全准确的。

该装饰器也提供了一个用于清理/使缓存失效的函数 `cache_clear()` 。

原始的未经装饰的函数可以通过 `__wrapped__` 属性访问。它可以用于检查、绕过缓存，或使用不同的缓存再次装饰原始函数。

[LRU（最久未使用算法）缓存](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)) 在最近的调用是即将到来的调用的最佳预测值时性能最好（例如，新闻服务器上最热门文章倾向于每天更改）。 缓存的大小限制可确保缓存不会在长期运行进程如网站服务器上无限制地增长。

下面我们再看下 `_lru_cache_wrapper` 的具体实现：

```
def _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo):
    # Constants shared by all lru cache instances:
    sentinel = object()          # unique object used to signal cache misses
    make_key = _make_key         # build a key from the function arguments
    PREV, NEXT, KEY, RESULT = 0, 1, 2, 3   # names for the link fields

    cache = {}
    hits = misses = 0
    full = False
    cache_get = cache.get    # bound method to lookup a key or return None
    cache_len = cache.__len__  # get cache size without calling len()
    lock = RLock()           # because linkedlist updates aren't threadsafe
    root = []                # root of the circular doubly linked list
    root[:] = [root, root, None, None]     # initialize by pointing to self

    if maxsize == 0:

        def wrapper(*args, **kwds):
            # No caching -- just a statistics update
            nonlocal misses
            misses += 1
            result = user_function(*args, **kwds)
            return result

    elif maxsize is None:

        def wrapper(*args, **kwds):
            # Simple caching without ordering or size limit
            nonlocal hits, misses
            key = make_key(args, kwds, typed)
            result = cache_get(key, sentinel)
            if result is not sentinel:
                hits += 1
                return result
            misses += 1
            result = user_function(*args, **kwds)
            cache[key] = result
            return result

    else:

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
                misses += 1
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
            return result

    def cache_info():
        """Report cache statistics"""
        with lock:
            return _CacheInfo(hits, misses, maxsize, cache_len())

    def cache_clear():
        """Clear the cache and cache statistics"""
        nonlocal hits, misses, full
        with lock:
            cache.clear()
            root[:] = [root, root, None, None]
            hits = misses = 0
            full = False

    wrapper.cache_info = cache_info
    wrapper.cache_clear = cache_clear
    return wrapper
```

函数根据 `maxsize` 不同，定义不同的 `wrapper`。

- `maxsize == 0`，其实也就是没有缓存，那么每次函数调用都不会命中，并且没有命中的次数 `misses` 加 1。
- `maxsize is None`，不限制缓存大小，如果函数调用不命中，将没有命中次数 `misses` 加 1，否则将命中次数 `hits` 加 1。
- 限制缓存的大小，那么需要根据 LRU 算法来更新 `cache`
  - 如果缓存命中 key，那么将命中节点移到双向循环链表的结尾，`hits += 1` ，并且返回结果。**这里通过字典加双向循环链表的组合数据结构，实现了用 O(1) 的时间复杂度删除给定的节点。**
  - 如果没有命中，并且缓存满了，那么需要将最久没有使用的节点（root 的下一个节点）删除，并且将新的节点添加到链表结尾。在实现中有一个优化，直接将当前的 root 的 key 和 result 替换成新的值，将 root 的下一个节点置为新的 root，这样得到的双向循环链表结构跟删除 root 的下一个节点并且将新节点加到链表结尾是一样的，但是避免了删除和添加节点的操作
  - 如果没有命中，并且缓存没满，那么直接将新节点添加到双向循环链表的开头

最后给 `wrapper` 添加两个属性函数 `cache_info` 和 `cache_clear`，`cache_info` 显示当前缓存的命中情况的统计数据，`cache_clear` 用于清空缓存。

最后需要说明的是，**对于有多个关键字参数的函数，如果两次调用函数关键字参数传入的顺序不同，会被认为是不同的调用，不会命中缓存。另外，被 lru_cache 装饰的函数不能包含可变类型参数如 list，因为它们不支持 hash。**



Read More:

> [functools --- 高阶函数和可调用对象上的操作](https://docs.python.org/zh-cn/3/library/functools.html)
>
> [Python 中 lru_cache 的使用和实现](https://www.cnblogs.com/zikcheng/p/14322577.html)