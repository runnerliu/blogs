## Python系列 - librdkafka 的安装和使用

[kafka](https://kafka.apache.org/) 是一种高吞吐量的分布式发布订阅消息系统。现在它已被 [多家不同类型的公司](https://cwiki.apache.org/confluence/display/KAFKA/Powered+By) 作为多种类型的数据管道和消息系统使用。Python对kafka的操作库主要有 [pykafka](https://github.com/Parsely/pykafka) 、[confluent-kafka-python](https://github.com/confluentinc/confluent-kafka-python) 、[librdkafka](https://github.com/edenhill/librdkafka) ，pykafka是Python内置的kafka操作模块，纯Python编写，提供 [simpleconsumer](http://pykafka.readthedocs.io/en/latest/api/simpleconsumer.html) 、[balancedconsumer](http://pykafka.readthedocs.io/en/latest/api/balancedconsumer.html) 两种消费方式。

其实严格意义上来说，librdkafka并不是kafka的操作库，它是Apache Kafka协议的C库实现，包含Producer和Consumer支持。

> **librdkafka** is a C library implementation of the [Apache Kafka](http://kafka.apache.org/) protocol, containing both Producer and Consumer support. It was designed with message delivery reliability and high performance in mind, current figures exceed 1 million msgs/second for the producer and 3 million msgs/second for the consumer.

由于项目的需要，原本使用pykafka作为数据的读取库，但是可能是达到了Python的读瓶颈，pykafka无法满足目前的要求，所以准备采用librdkafka来提高从kafka中读数据的效率。网上对Python环境中librdkafka的安装并没有详细的教程，通过自己的摸索和踩坑，终于是安装成功了。

### 安装librdkafka

执行

```
git clone https://github.com/edenhill/librdkafka.git
```

将librdkafka源码库下载到本地，如果没有安装git，请自行搜索安装。

如果github上无法下载，请转到百度网盘：链接: https://pan.baidu.com/s/1sl0uj2d 密码: af7k

下载的是0.11.1版本，本文安装的也是这个版本。

本文直接下载到/root/目录下，执行`cd librdkafka-0.11.1`进行目录

依次执行一下命令进行安装：

```
./configure
make && make install
```

librdkafka依赖的环境如下，如果未安装请自行安装

```
The GNU toolchain
GNU make
pthreads
zlib (optional, for gzip compression support)
libssl-dev (optional, for SSL and SASL SCRAM support)
libsasl2-dev (optional, for SASL GSSAPI support)
```

如果安装成功，会在`/usr/local/lib`目录中出现以下文件

```
[root@hcdn-others-worker-dev100-bjlt librdkafka-0.11.1]# cd /usr/local/lib
[root@hcdn-others-worker-dev100-bjlt lib]# ll
total 15200
-rwxr-xr-x 1 root root 8122982 Jan  4 15:21 librdkafka.a
-rwxr-xr-x 1 root root 2305704 Jan  4 15:21 librdkafka++.a
lrwxrwxrwx 1 root root      15 Jan  4 15:21 librdkafka.so -> librdkafka.so.1
lrwxrwxrwx 1 root root      17 Jan  4 15:21 librdkafka++.so -> librdkafka++.so.1
-rwxr-xr-x 1 root root 4225307 Jan  4 15:21 librdkafka.so.1
-rwxr-xr-x 1 root root  899617 Jan  4 15:21 librdkafka++.so.1
drwxr-xr-x 2 root root    4096 Jan  4 15:21 pkgconfig
```

librdkafka.so、librdkafka.so.1是librdkafka的动态链接库，因为Linux默认的动态链接库路径是`/lib`、`/usr/lib`，而librdkafka默认安装到了`/usr/local/lib`，为了使Python解释器可以找到librdkafka的动态链接库，本文采取的方法是修改系统文件`/etc/ld.so.conf`，这个文件中指定了默认的动态链接库查找路径，默认只有一行配置，如下

```
include ld.so.conf.d/*.conf
```

可以看到，`/etc/ld.so.conf`包含了ld.so.conf.d目录中的所有`.conf`文件，所以我们新建一个配置文件

```
[root@hcdn-others-worker-dev100-bjlt ld.so.conf.d]# ll
total 24
...
-rw-r--r--  1 root root  15 Jan  4 11:02 librdkafka.conf
...
```

在`librdkafka.conf`文件中将`/usr/local/lib`路径包含进去

```
/usr/local/lib
```

保存退出后执行以下命令更新动态链接库的搜索配置文件

```
ldconfig
```

通过以上操作，Python解释器就可以找到刚才安装的librdkafka的动态链接库了。

### 安装pykafka

Python可以通过pip工具安装pykafka

```
pip install pykafka
```

但是我们需要使用librdkafka扩展，使用pip工具是无法安装librdkafka的，所以我们需要源码编译安装pykafka。

执行

```
git clone https://github.com/Parsely/pykafka.git
```

下载pykafka的源码包到本地，这时别着急执行`./configure`，文档中介绍：要使用librdkafka扩展，需要确保头文件和共享库是Python可以找到它们的地方，无论是在构建扩展还是在运行时。

> PyKafka includes a C extension that makes use of librdkafka to speed up producer and consumer operation. To use the librdkafka extension, you need to make sure the header files and shared library are somewhere where python can find them, both when you build the extension (which is taken care of by `setup.py develop`) and at run time. Typically, this means that you need to either install librdkafka in a place conventional for your system, or declare `C_INCLUDE_PATH`, `LIBRARY_PATH`, and `LD_LIBRARY_PATH` in your shell environment to point to the installation location of the librdkafka shared objects. You can find this location with locate librdkafka.so.

我们已经将librdkafka的动态链接库加载到系统的搜索路径中，所以这个没问题，但是头文件在哪里？

在`pykafka/pykafka/rdkafka/_rd_kafkamodule.c`文件中有这样一行代码

```
#include <librdkafka/rdkafka.h>
```

所以我们需要在刚才下载的pykafka源码包中新建一个目录，将`rdkafka.h`放进去，保证我们在编译安装时能够找到它

```
[root@hcdn-others-worker-dev100-bjlt pykafka]# ll
total 128
...
drwxr-xr-x 2 root root  4096 Jan  4 14:22 librdkafka
...
```

`rdkafka.h`文件可以在刚才下载的librdkafka源码包中找到，即`librdkafka/src/`目录下

执行

```
cp /root/librdkafka-0.11.1/src/rdkafka.h /root/pykafka/librdkafka/
```

**注意**：/root代表librdkafka-0.11.1的存放目录

完成以上工作，依次执行

```
cd pykafka/
python setup.py build
python setup.py install
```

不出意外的话，就可以安装成功，这时在Python的安装目录的`site-packages/`目录下会产生`pykafka-2.7.0.dev2-py3.6-linux-x86_64.egg`目录，内容如下

```
[root@hcdn-others-worker-dev100-bjlt pykafka-2.7.0.dev2-py3.6-linux-x86_64.egg]# ll
total 12
drwxr-xr-x 2 root root 4096 Jan  4 15:22 EGG-INFO
drwxr-xr-x 7 root root 4096 Jan  4 15:22 pykafka
drwxr-xr-x 4 root root 4096 Jan  4 15:22 tests
```

表明安装成功了，在`pykafka/rdkafka/`目录中可以看到

```
[root@hcdn-others-worker-dev100-bjlt rdkafka]# ll
total 172
-rw-r--r-- 1 root root  1188 Jan  4 15:22 helpers.py
-rw-r--r-- 1 root root    89 Jan  4 15:22 __init__.py
-rw-r--r-- 1 root root  8343 Jan  4 15:22 producer.py
drwxr-xr-x 2 root root  4096 Jan  4 16:07 __pycache__
-rwxr-xr-x 1 root root 83815 Jan  4 15:22 _rd_kafka.cpython-36m-x86_64-linux-gnu.so
-rw-r--r-- 1 root root 41405 Jan  4 15:22 _rd_kafkamodule.c
-rw-r--r-- 1 root root   314 Jan  4 15:22 _rd_kafka.py
-rw-r--r-- 1 root root 12674 Jan  4 16:04 simple_consumer.py
```

### 踩的坑

- 使用执行报错1

```
pykafka.rdkafka#consumer-1 [PROTOERR] [thrd:openlive-kafka-online005-bjlt.qiyi.virtual:9092/bootstrap]: openlive-kafka-online005-bjlt.qiyi.virtual:9092/5: expected 36732025 bytes > 1048618 remaining bytes
pykafka.rdkafka#consumer-1 [PROTOERR] [thrd:openlive-kafka-online005-bjlt.qiyi.virtual:9092/bootstrap]: openlive-kafka-online005-bjlt.qiyi.virtual:9092/5: Protocol parse failure at 8/1048626 (rd_kafka_fetch_reply_handle:2496) (incorrect broker.version.fallback?)
```

错误给出的提示是`incorrect broker.version.fallback?`，一番google后，找到了问题所在，请看 [Broker version compatibility](https://github.com/edenhill/librdkafka/wiki/Broker-version-compatibility) ，librdkafka不同版本的broker对程序的配置要求不同，主要是`api.version.request` 、`broker.version.fallback`这两个参数的配置，在`pykafka/rdkafka/simple_consumer.py`文件中有这样的代码：

```
...
ver10 = parse_version(self._broker_version) >= parse_version("0.10.0")
...
conf = {
    ...
    "api.version.request": ver10,
    ...
}
...
if not ver10:
    conf["broker.version.fallback"] = self._broker_version
...
```

所以在使用时加上了`broker_version`参数，这个参数需要知道**kafka集群的broker的版本**，这个非常重要！

```
self.client = KafkaClient(zookeeper_hosts=zookeeper_hosts, broker_version="0.8.2")
```

这样使用就没有问题了。

但是在实际测试中，裸pykafka和pykafka-with-librdkafka的读性能并没有差太多，以下是我在实际中测试从kafka中读取50w条数据的耗时比较：

| pykafka            | pykafka-with-librdkafka |
| ------------------ | ----------------------- |
| 43.136744260787964 | 35.90807628631592       |
| 44.06431531906128  | 37.58351707458496       |
| 43.47287678718567  | 37.0159432888031        |

理论上，使用了C库的读取，其性能应该有很大提升才对，但是测试表明性能提升并不到，这个问题还有待探讨。