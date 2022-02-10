## Apache 2.4 配置虚拟主机

在开发项目的时候，每个项目对应的工程目录是不同的，想直接用一个自定义主机名来分别对应，这样就不用每次敲目录名了。Apache的虚拟主机可以实现这种功能。

配置虚拟主机有三种方式：

- 基于IP地址
- 基于主机名
- 基于端口号

本文主要介绍 **基于主机名** 和 **基于端口号** 这两种方式。

### 基于主机名

我们安装好Apache后，如果成功安装，在浏览器中中输入 localhost 即可显示出欢迎界面，Apache 默认的监听端口是80端口，这里不用输入80端口号就可以。

首先修改 *C:\Windows\System32\drivers\etc* 目录下的 hosts 文件，如果修改时保存失败，第一种方式是可以修改 hosts 文件的安全属性，右键->属性->安全，赋予相应用户的权限即可；第二种方式是，使用管理员身份打开记事本，然后在记事本中打开 hosts 文件，进行如下修改：

```
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost
192.168.1.105 windows10.microdone.cn
127.0.0.1       myapp.com			// 自定义的主机名，名称不能与网上的域名冲突
```

hosts 文件配置好后，打开 *\Apache24\conf* 目录下的 httpd.conf 文件，修改如下：

```
# Virtual hosts
# Include conf/extra/httpd-vhosts.conf

将 # 去掉

# Virtual hosts
Include conf/extra/httpd-vhosts.conf
```

然后打开 *\Apache24\conf\extra* 目录下的 httpd-vhosts.conf 文件，修改如下：

```
<VirtualHost *:80>
    ServerAdmin YourName
    DocumentRoot "${SRVROOT}/htdocs/laravel/public"
    ServerName myapp.com
    ErrorLog "${SRVROOT}/htdocs/laravel/myapp.com-error.log"
    CustomLog "${SRVROOT}/htdocs/laravel/myapp.com-access.log" common
    <Directory "${SRVROOT}/htdocs/laravel/public">
    	Options Indexes FollowSymLinks MultiViews
		AllowOverride None
		Require all granted
	</Directory>
</VirtualHost>
```

DocumentRoot 对应虚拟主机主目录； 
ServerName 对应主机名； 
ErrorLog 对应错误日志存放路径； 
CustomLog 对应访问日志存放路径； 
其中的< Directory >< Directory />对应相应地设置信息。

最后，在浏览器中输入 mapp.com/index.php 就可以访问不同目录了。

### 基于端口号

打开 *\Apache24\conf* 目录下的 httpd.conf 文件，修改如下：

```
#
# Listen: Allows you to bind Apache to specific IP addresses and/or
# ports, instead of the default. See also the <VirtualHost>
# directive.
#
# Change this to Listen on specific IP addresses as shown below to 
# prevent Apache from glomming onto all bound IP addresses.
#
#Listen 12.34.56.78:80
Listen 80
Listen 8090		// 添加的apache监听的另一端口号，确保未被占用
```

然后打开 *\Apache24\conf\extra* 目录下的 httpd-vhosts.conf 文件，修改如下：

```
<VirtualHost *:8090>
    ServerAdmin YourName
    DocumentRoot "${SRVROOT}/htdocs/laravel/public"
    ServerName myapp.com
    ErrorLog "${SRVROOT}/htdocs/laravel/myapp.com-error.log"
    CustomLog "${SRVROOT}/htdocs/laravel/myapp.com-access.log" common
    <Directory "${SRVROOT}/htdocs/laravel/public">
    	Options Indexes FollowSymLinks MultiViews
		AllowOverride None
		Require all granted
	</Directory>
</VirtualHost>
```



以上通过两种方式实现了通过访问不同的主机名来访问不同目录，基于IP地址的方式请访问  [Apache 配置虚拟主机三种方式](http://www.cnblogs.com/hi-bazinga/archive/2012/04/23/2466605.html)  

