## 博客列表

* [学习路线](#学习路线)
* [概念](#概念)
* [前端](#前端)
* [后端](#后端)
   * [架构](#架构)
   * [操作系统](#操作系统)
      * [Linux](#linux)
      * [Windows](#windows)
   * [设计模式](#设计模式)
   * [网络](#网络)
   * [Web 开发](#web-开发)
   * [数据库](#数据库)
      * [Mysql](#mysql)
   * [缓存](#缓存)
      * [Redis](#redis)
   * [消息队列](#消息队列)
   * [Web服务器](#web服务器)
      * [Nginx](#nginx)
      * [Apache](#apache)
   * [微服务](#微服务)
   * [版本管理](#版本管理)
      * [Git](#git)
   * [语言](#语言)
      * [PHP](#php)
      * [Python](#python)
      * [Golang](#golang)
* [算法](#算法)



### 学习路线

- [学习路线 2020](articles/roadmap-2020.md)

### 概念

- [抽象类和接口的区别](articles/concept-abstract-interface.md)
- [阻塞|非阻塞 - 异步|同步](articles/concept-ze-fze.md)
- [子进程和线程的区别](articles/concept-thread-subprocess.md)
- [进程与线程](articles/concept-process-thread.md)

### 前端

- [Datatables - 基础配置](articles/datatables.md)

### 后端

#### 架构

- [DDD系列 - 资料汇总](articles/ddd-learn.md)
- [秒杀系统设计 01 - 秒杀系统架构设计都有哪些关键点？](articles/miaosha-01.md)
- [秒杀系统设计 02 - 设计秒杀系统时应该注意的5个架构原则](articles/miaosha-02.md)
- [秒杀系统设计 03 - 如何才能做好动静分离？有哪些方案可选？](articles/miaosha-03.md)
- [秒杀系统设计 04 - 二八原则：有针对性地处理好系统的“热点数据”](articles/miaosha-04.md)
- [秒杀系统设计 05 - 流量削峰这事应该怎么做？](articles/miaosha-05.md)
- [秒杀系统设计 06 - 影响性能的因素有哪些？又该如何提高系统的性能？](articles/miaosha-06.md)
- [秒杀系统设计 07 - 秒杀系统“减库存”设计的核心逻辑](articles/miaosha-07.md)
- [秒杀系统设计 08 - 准备Plan B：如何设计兜底方案?](articles/miaosha-08.md)

#### 操作系统

- [进程调度算法](articles/process-scheduling.md)

##### Linux

- [用户态和内核态](articles/linux-yht-nht.md)
- [CentOS 编译安装 Python3](articles/linux-install-py3.md)
- [CentOS 系统时间校对](articles/linux-centos-sys-time.md)
- [Linux 命令 - cd](articles/linux-cd.md)
- [Linux 命令 - cp](articles/linux-cp.md)
- [Linux 命令 - crontab](articles/linux-crontab.md)
- [Linux 命令 - jq](articles/linux-jq.md)
- [Linux 命令 - ln](articles/linux-ln.md)
- [Linux 命令 - ls](articles/linux-ls.md)
- [Linux 命令 - netstat](articles/linux-netstat.md)
- [Linux 命令 - ping](articles/linux-ping.md)
- [Linux 命令 - ps](articles/linux-ps.md)
- [Linux 命令 - tar](articles/linux-tar.md)
- [Linux 命令 - top](articles/linux-top.md)
- [traceroute的实现原理](articles/linux-traceroute.md)

##### Windows

- [Windows 查看端口占用并关闭进程](articles/win-kill-process.md)

#### 设计模式

- [重构改善既有代码的设计 - 代码的坏味道](articles/bad-taste-of-code.md)
- [单例模式](articles/singleton.md)

#### 网络

- [HTTP 常用状态码详解](articles/http-status-code.md)
- [HTTP1.0、HTTP1.1、HTTP2.0的区别](articles/http-diff.md)
- [HTTPS 的加密原理](articles/https.md)
- [TCP/IP协议](articles/tcpip.md)
- [URL 请求过程](articles/url-request-process.md)
- [I/O 多路复用](articles/io-multiplexing.md)
- [聊聊WS和WSS](articles/ws-wss.md)
- [聊聊 WebSocket 与 Socket.IO](articles/websocket-socketio.md)
- [Websocket 与 Socket 的区别](articles/websocket-socket.md)

#### Web 开发

- [HTTP POST 数据提交方式](articles/http-post-method.md)
- [Session 共享解决方案](articles/share-session.md)

#### 数据库

##### Mysql

- [Mysql 系列 - 锁](articles/mysql-lock.md)
- [Mysql 系列 - 索引](articles/mysql-index.md)
- [Mysql 系列 - InnoDB 与 MyISAM](articles/mysql-engine.md)
- [Mysql 系列 - 数据类型](articles/mysql-datatype.md)
- [Mysql 系列 - explain](articles/mysql-explain.md)
- [Mysql 系列 - 回表查询和覆盖索引](articles/mysql-huibiao.md)
- [Mysql 系列 - 事务](articles/mysql-transaction.md)
- [Mysql 索引 - UNIQUE KEY 与 PRIMARY KEY](articles/)

#### 缓存

##### Redis

- [Redis 系列 - 缓存雪崩、击穿、穿透、预热、更新](articles/redis-cachedown.md)
- [Redis 系列 - 数据持久化](articles/redis-chijiuhua.md)
- [Redis 系列 - 过期策略和淘汰策略](articles/redis-gq-ttcl.md)
- [Redis 系列 - 主从切换](articles/redis-msswitch.md)
- [Redis 系列 - 主从同步](articles/redis-mssync.md)
- [Redis 系列 - 事务机制](articles/redis-transactions.md)

#### 消息队列

- [MQ 如何处理消息丢失](articles/mq-msg-lost.md)
- [Kafka 文件存储机制](articles/kafka-save.md)

#### Web服务器

##### Nginx

- [Nginx 系列 - 开机启动](articles/nginx-start-poweron.md)

##### Apache

- [Apache 2.4 配置虚拟主机](articles/apache-virtualhost.md)

#### 微服务

- [分布式锁](articles/distributed-lock.md)

#### 版本管理

##### Git

- [Git 常用命令](articles/git-command.md)

#### 语言

##### PHP

- [PHP 系列 - 垃圾回收机制](articles/php-gc.md)
- [PHP 系列 - FCGI 与 FPM](articles/php-fpm-fcgi.md)
- [PHP 系列 - $_SERVER](articles/php-server.md)
- [PHP 系列 - self,final,static,this,parent](articles/php-keywords.md)
- [PHP 系列 - 变量的内部存储（值和类型）](articles/php-varinternstore-01.md)
- [PHP 系列 - 变量的内部存储（引用和计数）](articles/php-varinternstore-02.md)
- [Laravel 系列 - 后台用户认证](articles/php-laravel-adminlogin.md)
- [Laravel 系列 - 用户登录实现及注册源码初探](articles/php-laravel-userlogin.md)
- [Laravel 系列 - 理解控制反转(IoC)和依赖注入(DI)](articles/php-laravel-ioc-di.md)

##### Python

- [Python 库 - requests 会话对象](articles/py-requests-session.md)
- [Python 库 - functools lru_cache](articles/py-functools-cache.md)
- [Python 库 - celery](articles/py-celery.md)
- [Python 库 - gevent](articles/py-gevent.md)
- [Python 库 - watchdog](articles/py-watchdog.md)
- [Python 库 - Transitions](articles/py-transitions.md)
- [Python 库 - SQLAlchemy 事务异常](articles/py-sqlalchemy-01.md)
- [Python 库 - isort](articles/py-isort.md)
- [Python 系列 - 上下文管理器](articles/py-contextmanager.md)
- [Python 系列 - 垃圾回收机制](articles/py-gc.md)
- [Python 系列 - 计算密集型任务和 I/O 密集型任务](articles/py-jsmi-iomiji.md)
- [Python 系列 - librdkafka 的安装和使用](articles/librdkafka.md)
- [Python 系列 - 项目部署 Nginx + Tornado + Supervisor](articles/py-supervisor-nginx.md)
- [Tornado5.0.2 翻译文档 - Tornado](articles/tornado502-start.md)

##### Golang

- [GO 系列 - 学习资料](articles/go-series.md)
- [GO 系列 - 数据类型](articles/go-datatype.md)
- [GO 系列 - 垃圾回收机制](articles/go-gc.md)
- [GO 库 - ants](articles/go-lib-ants.md)

### 算法

- [排序算法总结](articles/sort-algorithm.md)
- [约瑟夫环](articles/joseph-ring.md)