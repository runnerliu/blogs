## Tornado5.0.2 翻译文档 - 协程

Tornado中推荐使用 **协程** 编写异步代码，协程使用了Python的 `yield` 关键字代替链式回调来将程序挂起和恢复执行(像在 [gevent](http://www.gevent.org/) 中出现的轻量级线程合作方式有时也被称为协程，但是在Tornado中所有的协程使用明确的上下文切换，并被称为异步函数)。

使用协程几乎像写同步代码一样简单，并且不需要浪费额外的线程。它们还通过减少上下文切换来 [使并发编程更简单](https://glyph.twistedmatrix.com/2014/02/unyielding.html) 。

例子：

```
from tornado import gen

@gen.coroutine
def fetch_coroutine(url):
    http_client = AsyncHTTPClient()
    response = yield http_client.fetch(url)
    # In Python versions prior to 3.3, returning a value from
    # a generator is not allowed and you must use
    #   raise gen.Return(response.body)
    # instead.
    return response.body
```

### Python 3.5: `async` 和 `await`

Python 3.5 引入了 `async` 和 `await` 关键字(使用这些关键字的函数也被称为”原生协程”)。从Tornado 4.3，你可以用它们代替 `yield` 为基础的协程。只需要简单的使用 `async def foo()` 在函数定义的时候代替 `@gen.coroutine` 装饰器，用 `await` 代替 yield。本文档的其他部分会继续使用 `yield` 的风格来和旧版本的Python兼容，但是如果 `async` 和 `await` 可用的话，它们运行起来会更快：

```
async def fetch_coroutine(url):
    http_client = AsyncHTTPClient()
    response = await http_client.fetch(url)
    return response.body
```

`await` 关键字比 `yield` 关键字功能要少一些。例如，在一个使用 `yield` 的协程中，你可以得到 `Futures` 列表，但是在原生协程中，你必须把列表用 [`tornado.gen.multi`](http://tornado-zh.readthedocs.io/zh/latest/gen.html#tornado.gen.multi) 包起来。同时也去掉了与 [`concurrent.futures`](https://docs.python.org/3.6/library/concurrent.futures.html#module-concurrent.futures) 的集成。你也可以使用 [`tornado.gen.convert_yielded`](http://tornado-zh.readthedocs.io/zh/latest/gen.html#tornado.gen.convert_yielded) 来把任何使用 `yield` 工作的代码转换成使用 `await` 的形式：

```
async def f():
    executor = concurrent.futures.ThreadPoolExecutor()
    await tornado.gen.convert_yielded(executor.submit(g))
```

### 它如何工作

包含了 `yield` 关键字的函数是一个 **生成器(generator)** 。所有的生成器都是异步的；当调用它们的时候，会返回一个生成器对象，而不是一个执行完的结果。 `@gen.coroutine` 装饰器通过 `yield` 表达式和生成器进行交流，而且通过返回一个 [`Future`](http://tornado-zh.readthedocs.io/zh/latest/concurrent.html#tornado.concurrent.Future) 与协程的调用方进行交互。

下面是一个协程装饰器内部循环的简单版本：

```
# Simplified inner loop of tornado.gen.Runner
def run(self):
    # send(x) makes the current yield return x.
    # It returns when the next yield is reached
    future = self.gen.send(self.next)
    def callback(f):
        self.next = f.result()
        self.run()
    future.add_done_callback(callback)
```

装饰器从生成器接收一个 [`Future`](http://tornado-zh.readthedocs.io/zh/latest/concurrent.html#tornado.concurrent.Future) 对象，等待(非阻塞的)这个 [`Future`](http://tornado-zh.readthedocs.io/zh/latest/concurrent.html#tornado.concurrent.Future) 对象执行完成，然后“解开(unwraps)” 这个 [`Future`](http://tornado-zh.readthedocs.io/zh/latest/concurrent.html#tornado.concurrent.Future) 对象，并把结果作为 `yield` 表达式的结果传回给生成器。大多数异步代码从来不会直接接触 [`Future`](http://tornado-zh.readthedocs.io/zh/latest/concurrent.html#tornado.concurrent.Future) 类 除非 [`Future`](http://tornado-zh.readthedocs.io/zh/latest/concurrent.html#tornado.concurrent.Future) 立即通过异步函数返回给 `yield` 表达式。

### 如何调用协程

协程一般不会抛出异常：它们抛出的任何异常将被 [`Future`](http://tornado-zh.readthedocs.io/zh/latest/concurrent.html#tornado.concurrent.Future) 捕获直到它被得到，这意味着用正确的方式调用协程是重要的，否则你可能有被忽略的错误：

```
@gen.coroutine
def divide(x, y):
    return x / y

def bad_call():
    # This should raise a ZeroDivisionError, but it won't because
    # the coroutine is called incorrectly.
    divide(1, 0)
```

几乎所有的情况下，任何一个调用协程的函数都必须是协程它自身，并且在调用的时候使用 `yield` 关键字。当你重写超类中的方法，请参阅文档，看看协程是否支持(文档应该会写该方法 “可能是一个协程” 或者 “可能返回 一个 [`Future`](http://tornado-zh.readthedocs.io/zh/latest/concurrent.html#tornado.concurrent.Future) ”)：

```
@gen.coroutine
def good_call():
    # yield will unwrap the Future returned by divide() and raise
    # the exception.
    yield divide(1, 0)
```

有时你可能想要对一个协程”一劳永逸”而且不等待它的结果。在这种情况下，建议使用 [`IOLoop.spawn_callback`](http://tornado-zh.readthedocs.io/zh/latest/ioloop.html#tornado.ioloop.IOLoop.spawn_callback) ，它使得 [`IOLoop`](http://tornado-zh.readthedocs.io/zh/latest/ioloop.html#tornado.ioloop.IOLoop) 负责调用。如果它失败了， [`IOLoop`](http://tornado-zh.readthedocs.io/zh/latest/ioloop.html#tornado.ioloop.IOLoop) 会在日志中把调用栈记录下来：

```
# The IOLoop will catch the exception and print a stack trace in
# the logs. Note that this doesn't look like a normal call, since
# we pass the function object to be called by the IOLoop.
IOLoop.current().spawn_callback(divide, 1, 0)
```

如果函数使用了 `@gen.coroutine` ，则推荐以上方式使用 [`IOLoop.spawn_callback`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop.spawn_callback) ，但如果函数使用 `async def` ，则必须使用以上方式(否则协程不会运行)。

最后，在程序顶层，如果 IOLoop 尚未运行，你可以启动 [`IOLoop`](http://tornado-zh.readthedocs.io/zh/latest/ioloop.html#tornado.ioloop.IOLoop) 执行协程，然后使用 [`IOLoop.run_sync`](http://tornado-zh.readthedocs.io/zh/latest/ioloop.html#tornado.ioloop.IOLoop.run_sync) 方法停止 [`IOLoop`](http://tornado-zh.readthedocs.io/zh/latest/ioloop.html#tornado.ioloop.IOLoop) 。这通常被用来启动面向批处理程序的 `main` 函数：

```
# run_sync() doesn't take arguments, so we must wrap the
# call in a lambda.
IOLoop.current().run_sync(lambda: divide(1, 0))
```

### 协程模式

#### 调用阻塞函数

从一个协程调用阻塞函数最简单的方式是使用 `IOLoop.run_in_executor` ，它将返回和协程兼容的`Futures` ：

```
@gen.coroutine
def call_blocking():
    yield IOLoop.current().run_in_executor(blocking_func, args)
```

#### 并行

协程装饰器能识别列表或者字典对象中各自的 `Futures` ，并且并行的等待这些 `Futures` ：

```
@gen.coroutine
def parallel_fetch(url1, url2):
    resp1, resp2 = yield [http_client.fetch(url1),
                          http_client.fetch(url2)]

@gen.coroutine
def parallel_fetch_many(urls):
    responses = yield [http_client.fetch(url) for url in urls]
    # responses is a list of HTTPResponses in the same order

@gen.coroutine
def parallel_fetch_dict(urls):
    responses = yield {url: http_client.fetch(url)
                        for url in urls}
    # responses is a dict {url: HTTPResponse}
```

如果使用 `await` 关键字，列表和字典必须使用 [`tornado.gen.multi`](http://www.tornadoweb.org/en/stable/gen.html#tornado.gen.multi) 包装起来：

```
async def parallel_fetch(url1, url2):
    resp1, resp2 = await gen.multi([http_client.fetch(url1),
                                    http_client.fetch(url2)])
```

#### 交叉存取

有时候我们需要保存`Future` 而不是立即返回，所以可以在等待之前执行其他操作：

```
@gen.coroutine
def get(self):
    fetch_future = self.fetch_next_chunk()
    while True:
        chunk = yield fetch_future
        if chunk is None: break
        self.write(chunk)
        fetch_future = self.fetch_next_chunk()
        yield self.flush()
```

这种模式最适用于 `@gen.coroutine` ，如果 `fetch_next_chunk()` 使用 `async def` ，则必须如下调用： `fetch_future = tornado.gen.convert_yielded(self.fetch_next_chunk())`才能启动后台进程。

#### 循环

在原生协程中，可以使用`aysnc for` 。在老版本的Python中，协程的循环是棘手的，因为没有办法在 `for` 循环或者 `while` 循环 `yield` 迭代器，并且捕获 yield 的结果。相反，你需要将循环条件从访问结果中分离出来，下面是一个使用 [Motor](http://motor.readthedocs.org/en/stable/) 的例子：

```
import motor
db = motor.MotorClient().test

@gen.coroutine
def loop_example(collection):
    cursor = db.collection.find()
    while (yield cursor.fetch_next):
        doc = cursor.next_object()
```

#### 在后台运行

[`PeriodicCallback`](http://tornado-zh.readthedocs.io/zh/latest/ioloop.html#tornado.ioloop.PeriodicCallback) 通常不使用协程。相反，一个协程可以包含一个 `while True:` 循环并使用[`tornado.gen.sleep`](http://tornado-zh.readthedocs.io/zh/latest/gen.html#tornado.gen.sleep) ：

```
@gen.coroutine
def minute_loop():
    while True:
        yield do_something()
        yield gen.sleep(60)

# Coroutines that loop forever are generally started with
# spawn_callback().
IOLoop.current().spawn_callback(minute_loop)
```

有时可能会遇到一个更复杂的循环。例如，上一个循环运行每次花费 `60+N` 秒，其中 `N` 是 `do_something()` 花费的时间。为了 准确的每60秒运行，使用上面的交叉模式：

```
@gen.coroutine
def minute_loop2():
    while True:
        nxt = gen.sleep(60)   # Start the clock.
        yield do_something()  # Run while the clock is ticking.
        yield nxt             # Wait for the timer to run out.
```



Read More:

> [Coroutines](http://www.tornadoweb.org/en/stable/guide/coroutines.html) 

