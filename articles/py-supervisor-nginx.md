## Python 系列 - 项目部署 Nginx + Tornado + Supervisor

在项目中应用到了 Tornado 框架，开始时只是单进程裸跑 Tornado 实例，但是在高并发的情况下，单进程无法满足上千的连接请求。Nginx 是我们熟知的反向代理服务器，它可以支持上万的平行连接，在与 PHP、Python 集成后可以应用在大规模的集群上，Nginx 同时也可以用作负载平衡器，以减小单点服务器的压力。

[Supervisor](http://supervisord.org/) 是一个进程管理工具，我们可以使用它管理多个 Tornado 实例，非常方便。

总的思路是：Nginx 通过唯一监听端口接收大量请求，然后将请求分发到不同的 Tornado 实例上，Supervisor 将多个 Tornado 实例进程统一管理。

### Tornado

Tornado 和现在的主流 Web 服务器框架（包括大多数 Python 的框架）有着明显的区别：它是非阻塞式服务器，而且速度相当快。得利于其非阻塞的方式和对 [epoll](http://kaimingwan.com/post/linux/epollshi-xian-yuan-li-qian-xi) 的运用，Tornado 每秒可以处理数以千计的连接，这意味着对于实时 Web 服务来说，Tornado 是一个理想的 Web 框架。Tornado 由于内置了支持 epoll/kqueue 等高效网络库，而具备了处理高并发的能力。

程序入口：

```
# -*- coding: utf-8 -*-
import os.path

import tornado.escape
import tornado.ioloop
import tornado.web
from tornado.httpserver import HTTPServer
from tornado.options import define, options

from handlers import stream_handler

define("port", default=8888, help="run on the given port", type=int)
define("debug", default=False, help="run in debug mode")


class Application(tornado.web.Application):
    def __init__(self):
        handlers = [
            # (r"/", main_handler.MainHandler),
            (r"/stream.*?", stream_handler.StreamHandler),
        ]
        settings = dict(
            template_path=os.path.join(os.path.dirname(__file__), "templates"),
            static_path=os.path.join(os.path.dirname(__file__), "static"),
            debug=options.debug,
        )
        super(Application, self).__init__(handlers, **settings)

    pass


def main():
    tornado.options.parse_command_line()
    http_server = HTTPServer(Application())
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()


if __name__ == "__main__":
    main()
```

执行以下命令运行：

```
python27 app.py --port=8081
```

**注意：** 因为我的机器上安装了不同版本的 Python 解释器，`python27` 是我为 Python 2.7.13 创建的软连接。

以下一行代码必须有：

```
tornado.options.parse_command_line()
```

因为我们运行程序脚本时，需要指定不同的端口，这是在执行 Tornado 的解析命令行。

### Supervisor 管理 Tornado 进程

Supervisor 的安装可以在官方文章中找到，可使用 `yum pip easy_insatll` 等方法。最简单的就是：

```
pip27 install supervisor 
```

**注意：** 因为我的机器上安装了不同版本的 pip 包，`pip27` 是我为 Python 2.7.13 使用的。

创建文件夹：

```
mkdir /etc/supervisor
```

创建配置文件：

```
echo_supervisord_conf > /etc/supervisor/supervisord.conf
```

修改 `/etc/supervisor/supervisord.conf` 文件内容，在文件结尾 `[include]` 节点处

```
; [include]
; files = relative/directory/*.ini

=>

[include]
files = conf.d/*.conf
```

保存并退出

在 `/etc/supervisor/` 下创建 `conf.d ` 文件夹，及 `ProjectName.conf` (以项目名称命名的，例如：tornados.conf)

执行：

```
vi tornados.conf
```

添加以下内容：

```
[group:tornados]
programs=tornado-0,tornado-1,tornado-2,tornado-3,tornado-4,tornado-5,tornado-6,tornado-7

[program:tornado-0]
# 执行的命令
command=python27 /home/ly/analysisweb/app.py --port=8020
# 项目目录
directory=/home/ly/analysisweb/
# 用户
user=root
# 自动重启
autorestart=true
redirect_stderr=true
# log文件路径
stdout_logfile=/home/ly/analysisweb/log/supervisor/tornado0.log
# log级别
loglevel=info

[program:tornado-1]
command=python27 /home/ly/analysisweb/app.py --port=8021
directory=/home/ly/analysisweb/
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/home/ly/analysisweb/log/supervisor/tornado1.log
loglevel=info

[program:tornado-2]
command=python27 /home/ly/analysisweb/app.py --port=8022
directory=/home/ly/analysisweb/
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/home/ly/analysisweb/log/supervisor/tornado2.log
loglevel=info

[program:tornado-3]
command=python27 /home/ly/analysisweb/app.py --port=8023
directory=/home/ly/analysisweb/
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/home/ly/analysisweb/log/supervisor/tornado3.log
loglevel=info

[program:tornado-4]
command=python27 /home/ly/analysisweb/app.py --port=8024
directory=/home/ly/analysisweb/
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/home/ly/analysisweb/log/supervisor/tornado4.log
loglevel=info

[program:tornado-5]
command=python27 /home/ly/analysisweb/app.py --port=8025
directory=/home/ly/analysisweb/
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/home/ly/analysisweb/log/supervisor/tornado5.log
loglevel=info

[program:tornado-6]
command=python27 /home/ly/analysisweb/app.py --port=8026
directory=/home/ly/analysisweb/
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/home/ly/analysisweb/log/supervisor/tornado6.log
loglevel=info

[program:tornado-7]
command=python27 /home/ly/analysisweb/app.py --port=8027
directory=/home/ly/analysisweb/
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/home/ly/analysisweb/log/supervisor/tornado7.log
loglevel=info
```

保存并退出

执行以下命令启动 Supervisor ：

```
supervisord -c /etc/supervisor/supervisord.conf
```

> 坑1：出现错误 Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.For help, use /usr/bin/supervisord –h
>
> 解决办法：是因为有一个使用 supervisor 配置的应用程序正在运行。执行 `ps -ef | grep supervisor` 查看进程，杀掉 supervisor 的所有进程，重新运行 [参考这里](https://stackoverflow.com/questions/25121838/supervisor-on-debian-wheezy-another-program-is-already-listening-on-a-port-that) 
>
> 坑2：出现错误 Unlinking stale socket /tmp/supervisor.sock 
>
> 解决办法：执行 `unlink /tmp/supervisor.sock` [参考这里](https://stackoverflow.com/questions/14479894/stopping-supervisord-shut-down) 

如果执行成功，`ps -ef | grep python` 结果如下：

```
root     21314     1  0 15:22 ?        00:00:00 /usr/bin/python27 /usr/local/python2.7/bin/supervisord -c /etc/supervisor/supervisord.conf
root     21315 21314  0 15:22 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8021
root     21316 21314  0 15:22 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8020
root     21317 21314  0 15:22 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8023
root     21318 21314  0 15:22 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8022
root     21319 21314  0 15:22 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8025
root     21320 21314  0 15:22 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8024
root     21321 21314  0 15:22 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8027
root     21322 21314  0 15:22 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8026
```

### Nginx 作反向代理

![2017-9-1 161933](../images/2017-9-1 161933.jpg)

安装好 Nginx 后，编辑 nginx.conf 文件：

```
http {
    # 我启动了8个Tornado实例，配置不同端口
    upstream tornados {
        server 127.0.0.1:8020;
        server 127.0.0.1:8021;
        server 127.0.0.1:8022;
        server 127.0.0.1:8023;
        server 127.0.0.1:8024;
        server 127.0.0.1:8025;
        server 127.0.0.1:8026;
        server 127.0.0.1:8027;
    }
    
    proxy_next_upstream error;

    server {
        # nginx 监听8081端口
        listen       8081;
		
		# 静态文件的路径
        location /static/{
            alias /home/ly/analysisweb/static/;
            expires 24h;
        }

        location / {
            proxy_pass_header Server;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_pass http://tornados;
        }

    }

    server {
        ......
    }
}
```

可参考 [运行和部署](http://tornado-zh.readthedocs.io/zh/latest/guide/running.html) 

以上，就完成了Nginx+Tornado+Supervisor 的项目部署，分别启动 nginx、supervisord ，就可以达到不错的负载均衡效果（我的项目中没有访问静态资源的情景，所以主要还是作负载均衡用）