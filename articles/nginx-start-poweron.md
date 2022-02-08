## Nginx 系列 - 开机启动

Nginx 是一个很强大的高性能 Web 和反向代理服务器。虽然使用命令行可以对 nginx 进行各种操作，比如启动等，但是还是根据不太方便。下面介绍在 Linux 下安装后，如何设置开机自启动。

首先，在 Linux 系统的 ` /etc/init.d/` 目录下创建 nginx 文件，使用如下命令：

```
vim /etc/init.d/nginx
```

在脚本中添加如下命令：

```
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.2 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
#              It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
# 启动 nginx 需要执行的命令
nginxd=/usr/local/nginx/sbin/nginx
# nginx 的配置文件路径
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/var/run/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
   echo "nginx already running...."
   exit 1
fi
   echo -n $"Starting $prog: "
   daemon $nginxd -c ${nginx_config}
   RETVAL=$?
   echo
   [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
   return $RETVAL
}
# Stop nginx daemons functions.
stop() {
        echo -n $"Stopping $prog: "
        killproc $nginxd
        RETVAL=$?
        echo
        [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx.pid
}
# reload nginx service functions.
reload() {
    echo -n $"Reloading $prog: "
    #kill -HUP `cat ${nginx_pid}`
    killproc $nginxd -HUP
    RETVAL=$?
    echo
}
# See how we were called.
case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
reload)
        reload
        ;;
restart)
        stop
        start
        ;;
status)
        status $prog
        RETVAL=$?
        ;;
*)
        echo $"Usage: $prog {start|stop|restart|reload|status|help}"
        exit 1
esac
exit $RETVAL
```

以下是根据 nginx 具体安装路径填写的：

```
# 启动 nginx 需要执行的命令
nginxd=/usr/local/nginx/sbin/nginx
# nginx 的配置文件路径
nginx_config=/usr/local/nginx/conf/nginx.conf
```

保存脚本文件后设置文件的执行权限：

```
chmod a+x /etc/init.d/nginx
```

然后，就可以通过该脚本对 nginx 服务进行管理了：

```
/etc/init.d/nginx start
/etc/init.d/nginx stop
```

上面的方法完成了用脚本管理 nginx 服务的功能，但是还是不太方便，比如要设置 nginx 开机启动等。这时可以使用 chkconfig 来设置。

> chkconfig 命令主要用来更新（启动或停止）和查询系统服务的运行级信息。谨记 chkconfig 不是立即自动禁止或激活一个服务，它只是简单的改变了符号连接。

先将 nginx 服务加入 chkconfig 管理列表：

```
chkconfig --add /etc/init.d/nginx
```

加完这个之后，就可以使用 service 对 nginx 进行启动，重启等操作了：

```
service nginx start
service nginx stop
```

设置终端模式开机启动：

```
chkconfig nginx on
```