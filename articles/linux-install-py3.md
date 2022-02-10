## CentOS 编译安装 Python3

Linux 下大部分系统默认自带 python2.x 的版本，最常见的是 python2.6 或 python2.7 版本，默认的python 被系统很多程序所依赖，比如 CentOS 下的 yum 就是 python2.x 写的，所以默认版本不要轻易删除，否则会有一些问题。如果需要使用最新的 Python3 那么我们可以编译安装源码包到独立目录，这和系统默认环境之间是没有任何影响的，Python3 和 Python2 两个环境并存即可。

首先去 Python 官网下载 Python3 的源码包，[传送门](https://www.python.org/) 

进去之后点击导航栏的 Downloads，也可以鼠标放到 Downloads 上弹出菜单选择 Source code，表示源码包，这里选择最新版本 3.6.5，当然下面也有很多其他历史版本，点进去之后页面下方可以看到下载链接，包括源码包、Mac OSX 安装包、Windows 安装包等。Linux 系统可以直接使用以下命令：

```
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz
```

接下来，如果是全新的 Linux 系统，可能需要安装一些 Python 的依赖，如 openssl、readline等模块。需要的依赖主要如下：

```
yum -y install zlib zlib-devel
yum -y install bzip2 bzip2-devel
yum -y install ncurses ncurses-devel
yum -y install readline readline-devel
yum -y install openssl openssl-devel
yum -y install openssl-static
yum -y install xz lzma xz-devel
yum -y install sqlite sqlite-devel
yum -y install gdbm gdbm-devel
yum -y install tk tk-devel
```

如果不是全新的 Linux 系统，可以查看是否安装了以上依赖：

```
yum list installed | grep zlib
yum list installed | grep zlib-devel
yum list installed | grep bzip2
yum list installed | grep bzip2-devel
......
```

解压文件：

```
tar -zxvf Python-3.6.5.tgz
```

进入目录：

```
cd Python-3.6.5/
```

配置编译：

```
./configure --prefix=/usr/local/python3.6.5 --enable-shared CFLAGS=-fPIC
```

因为上面依赖包是用 yum 安装而不是自己编译的，所以都是安装在系统默认目录下，因此各种选项不用加默认即可生效。`--enable-shared` 和 `-fPIC` 之后可以将 Python3 的动态链接库编译出来，默认情况编译完 lib 下面只有 Python3.xm.a 这样的文件，python本身可以正常使用，但是如果编译第三方库需要python 接口的比如 caffe 等，则会报错；所以这里建议按照上面的方式配置。

然后编译并安装：

```
make && make install
```

整个过程大约5分钟，安装成功之后，安装目录就在 `/usr/local/python3.6.5/` 。

一般为了系统环境的干净，不需要将 python3、pip3 加入到系统环境变量中，使用全路径运行 Python 文件或安装第三方库即可，如：

```
/usr/local/python3.6.5/bin/python3.6 xxx.py
/usr/local/python3.6.5/bin/pip3.6 install twisted
```

但是一般在开发机上时，加入环境变量会更加方便，建立软连接如下：

```
ln -s /usr/python3.6.5/bin/python3.6 /usr/bin/python36
ln -s /usr/python3.6.5/bin/pip3 /usr/bin/pip3
```

然后可以直接使用软连接运行 Python 文件或安装第三方库：

```
python36 xxx.py
pip3 install twisted
```

如上在 CentOS 上安装 Python3 就完成了，安装其他版本的 Python 解释器同理。



Read More:

> [Linux下编译安装python3](https://www.cnblogs.com/freeweb/p/5181764.html)