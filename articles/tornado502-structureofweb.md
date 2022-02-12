## Tornado5.0.2 翻译文档 - Tornado web应用程序的结构

Tornado web应用程序通常由一个或多个 `RequestHandler` 子类组成，将传入请求路由到处理程序的 `Application` 对象，和一个启动服务程序的 `main` 方法。

一个最小的“hello world”示例是这样的:

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

### Application 对象

`Application` 对象负责全局配置，包括将请求映射到处理程序的路由表。

路由表是 `URLSpec` 对象(或元组)的列表，每个对象都包含(至少)一个正则表达式和一个处理程序类。顺序问题：使用第一个匹配规则。如果正则表达式包含捕获组，那么这些组就是路径参数，并将被传递给处理程序的HTTP方法。如果字典作为 `URLSpec` 的第三个元素传递，它将提供初始化参数，该参数将被传递给 `RequestHandler.initialize`。`URLSpec` 对象也可自定义名称，用于 `RequestHandler.reverse_url`。

例如，在这个片段中，根URL `/` 被映射到 `MainHandler`，而形式 `/story/` 后面跟着一个数字的 `URL` 被映射到 `StoryHandler`。这个数字被传递(作为字符串)给 `StoryHandler.get`。

```
class MainHandler(RequestHandler):
    def get(self):
        self.write('<a href="%s">link to story 1</a>' %
                   self.reverse_url("story", "1"))

class StoryHandler(RequestHandler):
    def initialize(self, db):
        self.db = db

    def get(self, story_id):
        self.write("this is story %s" % story_id)

app = Application([
    url(r"/", MainHandler),
    url(r"/story/([0-9]+)", StoryHandler, dict(db=db), name="story")
    ])
```

`Application` 构造函数接受许多关键字参数，这些参数可用于自定义应用程序的行为和启用可选特性；点击 [Application.settings](https://www.tornadoweb.org/en/stable/web.html#tornado.web.Application.settings)，查看完整设置。

### RequestHandler 子类

Tornado web应用程序的大部分工作都是在 `RequestHandler` 的子类中完成的。处理程序子类的主要入口点是以被处理的HTTP方法命名的方法: `get()`、`post()`等。每个处理程序可以定义一个或多个这样的方法来处理不同的HTTP操作。如上所述，这些方法将被调用，参数与匹配的路由规则的捕获组相对应。

在处理程序中，调用 `RequestHandler.render`、`RequestHandler.write`来产生一个响应。`render()` 按名称加载模板，并使用给定参数呈现它。`write()` 用于非基于模板的输出；它接受字符串、字节和字典(dicts将被编码为JSON)。

`RequestHandler` 中的许多方法被设计成可以在子类中重写，并在整个应用程序中使用。通常定义一个 `BaseHandler` 类，覆盖 `write_error` 和 `get_current_user` 等方法，然后为所有特定的处理程序子类化你自己的 `BaseHandler` 而不是 `RequestHandler`。

#### 处理请求输入

请求处理程序可以通过 `self.request` 访问表示当前请求的对象。有关属性的完整列表，请参阅 [HTTPServerRequest](https://www.tornadoweb.org/en/stable/httputil.html#tornado.httputil.HTTPServerRequest) 的类定义。

HTML表单使用的格式的请求数据将为您解析，并在 `get_query_argument` 和 `get_body_argument` 等方法中可用。

```
class MyFormHandler(tornado.web.RequestHandler):
    def get(self):
        self.write('<html><body><form action="/myform" method="POST">'
                   '<input type="text" name="message">'
                   '<input type="submit" value="Submit">'
                   '</form></body></html>')

    def post(self):
        self.set_header("Content-Type", "text/plain")
        self.write("You wrote " + self.get_body_argument("message"))
```

由于HTML表单编码对于参数是单个值还是包含一个元素的列表是模棱两可的，`RequestHandler` 提供了不同的方法，允许应用程序指示它是否需要列表。对于列表，使用 `get_query_arguments` 和 `get_body_arguments` 而不是它们的单一对应项。

通过表单上传的文件在 `self.request.files` 中可用。它将名称 `<input type="file">` 元素映射到一个文件列表。每个文件是一个字典形式 `{"filename":…, "content_type":……, "body":…}`。`files` 对象只有在文件上传带有表单包装器(即 `multipart/form-data` 类型)时才存在;如果未使用此格式，则在 `self.request.body` 中可以使用原始上传数据。默认情况下，上传的文件在内存中完全缓冲;如果您需要处理太大而不能舒适地保存在内存中的文件，请参阅 [stream_request_body](https://www.tornadoweb.org/en/stable/web.html#tornado.web.stream_request_body) 类装饰器。

在demos目录中，[file_receiver.py](https://github.com/tornadoweb/tornado/tree/master/demos/file_upload/)显示了两种接收文件上传的方法。

由于HTML表单编码的奇怪之处(例如，单参数与复数参数之间的模糊性)，Tornado并不试图将表单参数与其他类型的输入统一起来。特别地，我们不解析JSON请求体。希望使用JSON而不是form-encoding的应用程序可以重写 `prepare` 来解析它们的请求:

```
def prepare(self):
    if self.request.headers.get("Content-Type", "").startswith("application/json"):
        self.json_args = json.loads(self.request.body)
    else:
        self.json_args = None
```

### 重写RequestHandler方法

除了 `get()`/ `post()`/等，`RequestHandler` 中的某些其他方法被设计成在必要时由子类覆盖。在每个请求中，会发生以下调用序列:

 - 每个请求都会创建一个新的 `RequestHandler` 对象。
 - `initialize()` 使用来自应用程序配置的初始化参数调用。初始化通常应该只保存传递给成员变量的实参;它可能不会产生任何输出或调用 `send_error` 之类的方法。
 - `prepare()` 被调用。这在所有处理程序子类共享的基类中最有用，因为无论使用哪种HTTP方法都将调用 `prepare`。准备可产生产出;如果它调用`finish`(或`redirect`，等等)，处理将在此停止。
 - 其中一个HTTP方法被调用:`get()`、`post()`、`put()`等等。如果URL正则表达式包含捕获组，则将它们作为参数传递给此方法。
 - 当请求完成时，调用`on_finish()`。这通常是在`get()`或另一个HTTP方法返回之后。

所有设计要重写的方法都在`RequestHandler`文档中有这样的说明。一些最常见的覆盖方法包括:

 - `write_error` - 输出用于错误页面的HTML。
 - `on_connection_close` - 当客户端断开连接时调用;应用程序可以选择检测这种情况并停止进一步处理。请注意，不能保证能够迅速检测到关闭的连接。
 - `get_current_user` - 参见[User authentication](https://www.tornadoweb.org/en/stable/guide/security.html#user-authentication)。
 - `get_user_locale` - 返回用于当前用户的`Locale`对象。
 - `set_default_headers` - 可以用于设置响应的附加标头(例如自定义`Server`头)。

### 错误处理

如果处理程序抛出异常，Tornado将调用`RequestHandler.write_error`生成一个错误页面。`tornado.web.HTTPError`可以用来生成指定的状态码;所有其他异常返回一个500状态。

默认的错误页面包括调试模式下的堆栈跟踪和错误的单行描述(例如:“500: Internal Server Error”)。若要生成自定义错误页面，请覆盖`RequestHandler.write_error`(可能在所有处理程序共享的基类中)。该方法可以通过`write`和`render`等方法正常生成输出。如果错误是由异常引起的，那么将传递一个`exc_info`三元组作为关键字参数(注意，这个异常不能保证是`sys.exc_info`，所以`write_error`必须使用`traceback.format_exception`代替`traceback.format_exc`)。

通过调用`set_status`编写响应并返回，也可以从常规处理程序方法而不是`write_error`生成一个错误页面。在简便返回不很方便的情况下`tornado.web.Finish` 异常会被抛出来终止处理程序而不是抛出`write_error`。

对于404错误，使用`default_handler_class` `Application settings`。这个处理程序应该覆盖`prepare`而不是像`get()`这样更具体的方法，这样它就可以与任何HTTP方法一起工作。它应该像上面描述的那样生成它的错误页面:通过调用`HTTPError(404)`并覆盖`write_error`，或者调用`self.set_status(404)`并在`prepare()`中直接生成响应。

### 重定向

在Tornado中有两种主要的重定向请求的方法:`RequestHandler.redirect` 和 `RedirectHandler`。

您可以在`RequestHandler`方法中使用`self.redirect()`将用户重定向到其他地方。还有一个可选参数`permanent`，您可以使用它来指示重定向被认为是永久性的。`permanent`的默认值是False，它生成`302 Found HTTP`响应代码，适合在成功POST请求后重定向用户。如果`permanent`为True，则使用301移动的永久HTTP响应代码，这在以面向seo的方式重定向到页面的规范URL时很有用。

`RedirectHandler`允许您在应用程序路由表中直接配置重定向。例如，要配置一个静态重定向:

```
app = tornado.web.Application([
    url(r"/app", tornado.web.RedirectHandler,
        dict(url="http://itunes.apple.com/my-app-id")),
    ])
```
`RedirectHandler`也支持正则表达式替换。下面的规则将所有以`/pictures/`开头的请求重定向到前缀`/photos/`:

```
app = tornado.web.Application([
    url(r"/photos/(.*)", MyPhotoHandler),
    url(r"/pictures/(.*)", tornado.web.RedirectHandler,
        dict(url=r"/photos/{0}")),
    ])
```

不像`RequestHandler.redirect`，`RedirectHandler`默认使用永久重定向。这是因为路由表在运行时不会改变，并且被认为是永久的，而在处理程序中发现的重定向很可能是其他可能改变的逻辑的结果。要使用`RedirectHandler`发送临时重定向，请在`RedirectHandler`初始化参数中添加`permanent=False`。

### 异步处理类

某些处理程序方法(包括`prepare()`和HTTP动词方法`get()`/ `post()`/等)可能被重写为协程，以使处理程序异步。

例如，下面是一个使用协同程序的简单处理程序:

```
class MainHandler(tornado.web.RequestHandler):
    async def get(self):
        http = tornado.httpclient.AsyncHTTPClient()
        response = await http.fetch("http://friendfeed-api.com/v2/feed/bret")
        json = tornado.escape.json_decode(response.body)
        self.write("Fetched " + str(len(json["entries"])) + " entries "
                   "from the FriendFeed API")
```

关于更高级的异步示例，请查看[聊天示例应用程序](https://github.com/tornadoweb/tornado/tree/stable/demos/chat)，该应用程序使用长轮询实现了一个AJAX聊天室。使用长轮询的用户可能希望覆盖`on_connection_close()`，以便在客户端关闭连接后进行清理(但请参阅该方法的文档字符串以了解注意事项)。



Read More:

> [Structure of a Tornado web application](https://www.tornadoweb.org/en/stable/guide/structure.html)





