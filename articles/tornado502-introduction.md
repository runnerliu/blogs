## Tornado5.0.2 翻译文档 - 介绍

[Tornado](http://www.tornadoweb.org/) 是一个 Python web 框架和异步网络库，起初由 [FriendFeed](http://friendfeed.com/) 开发。通过使用非阻塞网络 I/O，Tornado 可以支持上万级的连接，处理[长连接](http://en.wikipedia.org/wiki/Push_technology#Long_polling) [WebSockets](http://en.wikipedia.org/wiki/WebSocket) 和其他需要与每个用户保持长久连接的应用。

Tornado 大体上可以被分为4个主要的部分：

- web 框架 - 包括创建 web 应用的 [`RequestHandler`]() 类，和其他支持类；
- HTTP 客户端和服务端的实现([`HTTPServer`]() 和 [`AsyncHTTPClient`]())；
- 异步网络库([`IOLoop`]() 和 [`IOStream`]())，为 HTTP 组件提供构建模块，也可以用来实现其他协议；
- 协程库 ([`tornado.gen`]()) 允许异步代码写的更直接而不用链式回调的方式。

Tornado web 框架和 HTTP server 一起为 [WSGI](http://www.python.org/dev/peps/pep-3333/) 提供了一个全栈式的选择，在 WSGI 容器 ([`WSGIAdapter`]()) 中使用 Tornado web 框架或者使用 Tornado HTTP server 作为一个其他 WSGI 框架([`WSGIContainer`]())的容器，这样的组合方式都是有局限性的。为了充分利用Tornado的特性，你需要一起使用 Tornado 的 web 框架和 HTTP server。



Read More:

> [Introduction](http://www.tornadoweb.org/en/stable/guide/intro.html) 



