## Tornado5.0.2 翻译文档 - 模板和UI

Tornado包含一种简单、快速、灵活的模板语言。本节描述该语言以及相关问题，如国际化。

Tornado还可以与任何其他Python模板语言一起使用，尽管没有将这些系统集成到`RequestHandler.render`中。只需将模板呈现为一个字符串并将其传递给`RequestHandler.write`即可。

### 模板配置

默认情况下，Tornado会在引用它们的`.py`文件的同一目录中寻找模板文件。要将模板文件放到不同的目录中，请使用`template_path`应用程序设置(或覆盖`RequestHandler.get_template_path`如果不同的处理程序有不同的模板路径)。

要从非文件系统位置加载模板，子类`tornado.template.BaseLoader`并传递一个实例作为`template_loader`应用程序设置。

编译后的模板默认缓存;要关闭此缓存并重新加载模板，以便底层文件的更改始终可见，请使用应用程序设置`compiled_template_cache=False`或`debug=True`。

### 模板语法

Tornado模板只是HTML(或任何其他基于文本的格式)，在标记中嵌入了Python控制序列和表达式:

```
<html>
   <head>
      <title>{{ title }}</title>
   </head>
   <body>
     <ul>
       {% for item in items %}
         <li>{{ escape(item) }}</li>
       {% end %}
     </ul>
   </body>
 </html>
```

如果您将此模板保存为“template.html”。然后把它放在与Python文件相同的目录下，你可以用下面的方法来渲染这个模板:

```
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        items = ["Item 1", "Item 2", "Item 3"]
        self.render("template.html", title="My title", items=items)
```

```
Tornado模板支持控制语句和表达式。控制语句被{%和%}包围，比如：{% if len(items) > 2 %}，表达式由{{和}}包围，比如{{ item[0] }}。控制语句或多或少精确地映射到Python语句。我们支持if、for、while和try，它们都以{%end%}结尾，我们还支持使用extends和block语句的模板继承，这些语句都能在tornado.template找到。
```

表达式可以是任何Python表达式，包括函数调用。模板代码在包含以下对象和函数的命名空间中执行(注意使用此列表呈现的模板`RequestHandler.render`和`render_string`。如果你使用`tornado.template`模块直接位于`RequestHandler`外部，其中许多条目不存在）


- `escape`: alias for `tornado.escape.xhtml_escape`

- `xhtml_escape`: alias for `tornado.escape.xhtml_escape`

- `url_escape`: alias for `tornado.escape.url_escape`

- `json_encode`: alias for `tornado.escape.json_encode`

- `squeeze`: alias for `tornado.escape.squeeze`

- `linkify`: alias for `tornado.escape.linkify`

- `datetime`: the Python `datetime`module

- `handler`: the current `RequestHandler` object

- `request`: alias for `handler.request`

- `current_user`: alias for `handler.current_user`

- `locale`: alias for `handler.locale`

- `\_`: alias for handler.locale.translate

- static_url: alias for `handler.static_url`

- xsrf_form_html: alias for `handler.xsrf_form_html`

- reverse_url: alias for `Application.reverse_url`

- All entries from the `ui_methods` and `ui_modules` `Application` settings

- Any keyword arguments passed to `render` or `render_string`


当您构建一个真正的应用程序时，您需要使用Tornado模板的所有特性，特别是模板继承。请阅读`tornado.template`部分（一些功能，包括`UIModules`在`tornado.web`模块）

Tornado模板直接翻译成Python。模板中包含的表达式将被逐字复制到表示模板的Python函数中。我们不试图阻止模板语言中的任何东西；我们显式地创建它是为了提供其他更严格的模板系统所阻止的灵活性。因此，如果您在模板表达式中编写随机内容，那么在执行模板时，您将得到随机的Python错误。

默认情况下，所有模板输出都是转义的，使用`tornado.escape.xhtml_escape`函数，可以传递参数 `autoescape=None` 给Application或者通过`tornado.template.Loader`来全局改变该属性，此外，在这些地方中的每一个都可以使用替代转义函数的名称来代替`None`。

请注意，虽然Tornado的自动转义有助于避免XSS漏洞，但它在所有情况下都不够。出现在某些位置（如JavaScript或CSS）中的表达式可能需要额外的转义。此外，必须注意在可能包含不可信内容的HTML属性中始终使用双引号和xhtml_escape，或者必须对属性使用单独的转义函数（参见[博客](https://wonko.com/post/html-escaping)）。

### 国际化

参见[Internationalization](https://www.tornadoweb.org/en/stable/guide/templates.html#internationalization)

### UI模块

参见[UI modules](https://www.tornadoweb.org/en/stable/guide/templates.html#ui-modules)




Read More:

> [Templates and UI](https://www.tornadoweb.org/en/stable/guide/templates.html)





