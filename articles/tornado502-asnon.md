## Tornado5.0.2翻译文档 - 异步和非阻塞I/O

实时web功能需要为每个用户提供一个多数时间被闲置的长连接，在传统的同步 web 服务器中，这意味着要为每个用户提供一个线程，当然每个线程的开销都是很昂贵的。

为了尽量减少并发连接造成的开销，Tornado 使用了一种单线程事件循环的方式。这就意味着所有的应用代码都应该是异步非阻塞的，因为在同一时间只有一个操作是有效的。

异步和非阻塞是非常相关的并且这两个术语经常交换使用，但它们不是完全相同的事情。

### 阻塞

一个函数在等待某些事情返回值的时候会被阻塞，导致函数阻塞的原因有很多：网络 I/O，磁盘 I/O，互斥锁等。事实上每个函数在运行和使用 CPU 的时候都或多或少被阻塞(举个极端的例子来说明为什么对待 CPU 阻塞要和对待一般阻塞一样的严肃：比如密码哈希函数 [bcrypt](http://bcrypt.sourceforge.net/) ，需要消耗几百毫秒的 CPU 时间，这已 经远远超过了一般的网络或者磁盘请求时间了)。

一个函数可以在某个方面被阻塞但在其他方面不被阻塞，在 Tornado 的上下文中，我们一般讨论网络 I/O 上下文的阻塞，尽管各种阻塞已经被最小化。

### 异步

异步函数在会在其完成之前返回，在应用中触发下一个动作之前通常会在后台执行一些工作(和正常的同步函数在返回前就执行完所有的事情不同)。这里列举了几种不同的异步接口：

- 回调参数；
- 返回一个占位符([`Future`]() [`Promise`]() [`Deferred`]())；
- 传送给一个队列；
- 回调注册表(POSIX 信号)。

不论使用哪种类型的接口，按照定义，异步函数与他们的调用者都有着不同的交互方式；也没有什么对调用者透明的方式使得同步函数变得异步(类似 [gevent](http://www.gevent.org/) 使用轻量级线程的系统性能虽然堪比异步系统，但它们并没有真正的让事情异步)。

### 例子

这是一个简单的同步函数：

```
from tornado.httpclient import HTTPClient

def synchronous_fetch(url):
    http_client = HTTPClient()
    response = http_client.fetch(url)
    return response.body
```

下面是使用回调参数重写的具有同样功能的异步函数：

```
from tornado.httpclient import AsyncHTTPClient

def asynchronous_fetch(url, callback):
    http_client = AsyncHTTPClient()
    def handle_response(response):
        callback(response.body)
    http_client.fetch(url, callback=handle_response)
```

使用 [`Future`]() 代替回调：

```
from tornado.concurrent import Future
from tornado.httpclient import AsyncHTTPClient

def async_fetch_future(url):
    http_client = AsyncHTTPClient()
    my_future = Future()
    fetch_future = http_client.fetch(url)
    fetch_future.add_done_callback(
        lambda f: my_future.set_result(f.result()))
    return my_future
```

[`Future`]() 版本更加复杂，但是 [`Futures`]() 却是 Tornado 中推荐的写法，因为它有两个主要的优势：一是错误处理更加一致，[`Future.result`]() 方法可以简单的抛出异常(相较于常见的回调函数接口中需特别指定错误处理方式)，二是 [`Futures`]() 很适合与协程一起使用。我们稍后会更加深入的讨论协程，以下是例子的协程版本，类似原始的同步版本：

```
from tornado import gen

@gen.coroutine
def fetch_coroutine(url):
    http_client = AsyncHTTPClient()
    response = yield http_client.fetch(url)
    raise gen.Return(response.body)
```

`raise gen.Return(response.body)` 声明是在 Python 2 环境下人为执行的，因为生成器不允许有返回值，为了解决这个问题，Tornado 的协程抛出了一种称为 `Return` 的异常，协程捕获这个异常并将其作为返回值。在 Python 3.3+版本，使用 `return response.body` 可以得到相同的结果。



Read More:

> [Asynchronous and non-Blocking I/O](http://www.tornadoweb.org/en/stable/guide/async.html)