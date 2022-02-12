## Tornado5.0.2翻译文档 - Tornado

[Tornado](http://www.tornadoweb.org/) 是一个 Python 的 Web 框架和异步网络库，最初由 [FriendFeed ](http://friendfeed.com/)开发。通过使用非阻塞网络I/O，Tornado 可以支持数以万计的连接，非常适合[长轮询](http://en.wikipedia.org/wiki/Push_technology#Long_polling)、[WebSockets](http://en.wikipedia.org/wiki/WebSocket) 和其他需要与每个用户建立长连接的应用程序。

### 快速链接

- 当前版本：5.0.2([download from PyPi](https://pypi.python.org/pypi/tornado) [release notes](http://www.tornadoweb.org/en/stable/releases.html))
- [源代码](https://github.com/tornadoweb/tornado)
- 邮件列表：[discussion](http://groups.google.com/group/python-tornado) 和 [announcements](http://groups.google.com/group/python-tornado-announce)
- [Stack Overflow](http://stackoverflow.com/questions/tagged/tornado)
- [Wiki](https://github.com/tornadoweb/tornado/wiki/Links)

### Hello, World

以下是利用 Tornado 实现的一个简单的 "Hello, World" 应用程序：

```
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello, world")

def make_app():
    return tornado.web.Application([
        (r"/", MainHandler),
    ])

if __name__ == "__main__":
    app = make_app()
    app.listen(8888)
    tornado.ioloop.IOLoop.current().start()
```

以上例子没有使用 Tornado 的任何异步特性，异步特性可以移步 [simple char room](https://github.com/tornadoweb/tornado/tree/stable/demos/chat)。

### 线程和 WSGI

Tornado 与其他大多 Python web 框架不同，它不基于 [WSGI](https://wsgi.readthedocs.io/en/latest/) 且每个进程只能运行一个 Tornado 线程，移步 [用户手册]() 了解更多 Tornado 的异步编程方式。

然而 WSGI 支持的一些方法可以在 [tornado.wsgi]() 模块中找到，但这并不是 Tornado 后续发展的重点，大多数 Tornado 应用应该直接使用其自带的接口（例如 [tornado.web]()）而不是 WSGI。

一般来说，Tornado 的应用代码并不是线程安全的，Tornado 中唯一可以安全地从其他线程调用的方法是 [IOLoop.add_callback]() 。你也可以使用 [IOLoop.run_in_executor]() 在另一个线程上异步运行阻塞函数，但请注意，传给 [run_in_executor]() 的函数不能引用任何 Tornado 对象。与阻塞函数进行交互时，推荐使用 [run_in_executor]() 。

### 安装

```
pip install tornado
```

Tornado 在 [PyPi](http://pypi.python.org/pypi/tornado) 可获取安装列表中，所以可以直接使用 `pip` 方式进行安装。请注意，源代码发行版包含演示应用程序，当以这种方式安装 Tornado 时，这些演示应用程序将不会存在，因此您可能希望下载源代码 tar 包或克隆 [git repository](https://github.com/tornadoweb/tornado)。

**安装条件**：Tornado 运行在 Python 2.7 和 Python 3.4+上。 Python 2.7.9 中需要对 ssl 模块进行更新（在某些发行版中，这些更新可能在较早的 Python 版本中可用）。除了 `pip` 或 `setup.py` 自动安装的依赖外，以下可选软件包可能会有用：

- [pycurl](http://pycurl.sourceforge.net/) 在 [tornado.curl_httpclient]() 中需要使用，需要 Libcurl 7.22或更高版本；
- [Twisted](http://www.twistedmatrix.com/) 在 [tornado.platform.twisted]() 的诸多类中会使用到；
- [pycares](https://pypi.python.org/pypi/pycares) 是一个可选的非阻塞 DNS 解析器，可以在线程不适用时使用；
- [monotonic](https://pypi.python.org/pypi/monotonic) 或 [Monotime](https://pypi.python.org/pypi/Monotime) 添加对单调时钟的支持，从而提高时钟频繁调整场景下的可靠性。 在Python 3中不再需要。

**平台**：尽管为了获得最佳性能和可扩展性，Tornado 应该可以在任何类Unix平台上运行，但对于生产部署，建议只使用Linux（with `epoll`）和 BSD（with `kqueue`）（尽管 Mac OS X 源自 BSD 并支持 kqueue，但其网络 性能一般很差，所以建议仅用于开发使用）。Tornado 也可以在 Windows 上运行，这种配置没有官方的支持，只推荐用于开发。 如果不修改 Tornado IOLoop 接口，就不可能添加本地 Tornado Windows IOLoop 实现，且不能利用像 AsyncIO 或 Twisted 等框架的 Windows 的 IOCP 支持。

### 文档

- [用户指南](https://runnerliu.github.io/2018/06/10/tornado502-userguide/#more)
  - [介绍](https://runnerliu.github.io/2018/06/10/tornado502-introduction/)
  - [异步和非阻塞 I/O](https://runnerliu.github.io/2018/06/10/tornado502-asnon/#more)
  - [协程](https://runnerliu.github.io/2018/06/18/tornado502-coroutines/#more)
  - [Queue 示例-一个并发网络爬虫](https://runnerliu.github.io/2018/06/18/tornado502-concurrentwebspider/#more)
  - [Tornado 的 web 应用程序结构](https://runnerliu.github.io/2020/12/19/tornado502-structureofweb/#more)
  - [模板和UI](https://runnerliu.github.io/2020/12/19/tornado502-templateui/#more)
  - [认证和安全]()
  - [运行和部署]()
- [Web 框架]()
  - [tornado.web - RequestHandler Application 类]()
  - [tornado.template - 灵活的输出生成]()
  - [tornado.routing - 基本的路由实现]()
  - [tornado.escape - 转义和字符串操作]()
  - [tornado.locale - 国际化支持]()
  - [tornado.websocket - 与浏览器的双向通信]()
- [HTTP 服务器和客户端]()
  - [tornado.httpserver - 非阻塞 HTTP 服务器]()
  - [tornado.httpclient - 异步 HTTP 客户端]()
  - [tornado.httputil - 操作HTTP标头和URL]()
  - [tornado.httpconnection - HTTP/1.x客户端/服务器实现]()
- [异步网络]()
  - [tornado.ioloop - 主时间循环]()
  - [tornado.iostream - 对非阻塞 socket 的简易包装]()
  - [tronado.netutil - 网络实用程序]()
  - [tornado.tcpclient - IOStream 连接工厂]()
  - [tornado.tcpserver - 基于 TCP 服务器的 IOStream]()
- [协程和并发]()
  - [tornado.gen - 基于生成器的协程]()
  - [tornado.locks - 同步原语]()
  - [tornado.queues - 协程队列]()
  - [tornado.process - 多进程的实用程序]()
- [与其他服务集成]()
  - [tornado.auth - 集成 OpenID 和 OAuth 的第三方登录服务]()
  - [tornado.wsgi - 与其他 Python 框架和服务器的互操作性]()
  - [tornado.platform.caresresolver - 使用 C-Ares 的异步DNS解析器]()
  - [tornado.platform.twisted - 兼容 Tornado 和 Twisted]()
  - [tornado.asyncio - 兼容 Tornado 和 asyncio]()
- [通用工具]()
  - [tornado.authoreload - 开发环境中自动检测代码变化]()
  - [tornado.concrrent - 与 Future 对象交互]()
  - [tornado.log - 日志支持]()
  - [tornado.options - 命令行解析]()
  - [tornado.stack_context - 异步回调异常处理]()
  - [tornado.testing - 异步代码的单元测试]()
  - [tornado.util - 通用程序]()

### 讨论和支持

你可以在 [the Tornado developer mailing list](http://groups.google.com/group/python-tornado) 参与 Tornado 的讨论，在 [GitHub issue tracker](https://github.com/tornadoweb/tornado/issues) 上提交你的bug，更多资源可以在 [Tornado wiki](https://github.com/tornadoweb/tornado/wiki/Links) 找到，新版本发布在 [announcements mailing list](http://groups.google.com/group/python-tornado-announce)。



Read More:

> [Tornado](http://www.tornadoweb.org/en/stable/index.html) 