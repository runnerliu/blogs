## Linux 命令 - ls

Linux ls命令用于显示指定工作目录下之内容（列出目前工作目录所含之文件及子目录），在Linux中是使用率较高的命令，ls命令的输出信息可以进行彩色加亮显示，以区分不同类型的文件。

#### 命令格式

```
ls [OPTION]... [FILE]...
```

#### 命令功能

列出目标目录中所有的子目录和文件。

#### 命令参数

| 参数                                      | 说明                                       |
| --------------------------------------- | ---------------------------------------- |
| -a                                      | 显示所有文件及目录（ls内定将文件名或目录名称开头为"."的视为隐藏档，不会列出） |
| -A                                      | 同 -a ，但不列出 "."（当前目录）及 ".."（父目录）          |
| -C                                      | 多列显示输出结果。这是默认选项                          |
| -l                                      | 除文件名称外，亦将文件型态、权限、拥有者、文件大小等资讯详细列出         |
| -F                                      | 在列出的文件名称后加一符号；例如可执行档则加 "*", 目录则加 "/"     |
| -c                                      | 与"-lt"选项连用时，按照文件状态时间排序输出目录内容，排序的依据是文件的索引节点中的ctime字段。与"-l"选项连用时，则排序的一句是文件的状态改变时间 |
| -d                                      | 仅显示目录名，而不显示目录下的内容列表。显示符号链接文件本身，而不显示其所指向的目录列表 |
| -f                                      | 此参数的效果和同时指定"aU"参数相同，并关闭"lst"参数的效果；       |
| -i                                      | 显示文件索引节点号（inode）。一个索引节点代表一个文件            |
| -k                                      | 以KB（千字节）为单位显示文件大小                        |
| -m                                      | 用","号区隔每个文件和目录的名称                        |
| -n                                      | 以用户识别码和群组识别码替代其名称                        |
| -r                                      | 将文件以相反次序显示(原定依英文字母次序)                    |
| -s                                      | 显示文件和目录的大小，以区块为单位                        |
| -t                                      | 将文件依建立时间之先后次序列出                          |
| -L                                      | 如果遇到性质为符号链接的文件或目录，直接列出该链接所指向的原始文件或目录     |
| -R                                      | 递归列出该目录中的所有文件                            |
| --full-time                             | 列出完整的日期与时间                               |
| --color[=WHEN] (WHEN=never/always/auto) | 使用不同的颜色高亮显示不同类型的                         |
| -g                                      | 类似 -l，但不列出所有者                            |
| -h                                      | 以容易理解的格式列出文件大小 (例如 1K 234M 2G)           |
| -S                                      | 根据文件大小排序                                 |
| -X                                      | 根据扩展名排序                                  |
| -x                                      | 逐行列出项目而不是逐栏列出                            |

#### 使用示例

- `ls -l ly`

```
[root@hcdn-others-worker-dev100-bjlt home]# ls -l ly
total 48
drwxr-xr-x  8 root root 4096 Dec 14 11:49 analysisweb
drwxr-xr-x  9 root root 4096 Dec 28 16:19 atsbtsp
drwxr-xr-x  2 root root 4096 Sep 12 14:37 auth_server_ratio
drwxr-xr-x  7 root root 4096 Jan  2 10:10 datawritehbase
drwxr-xr-x 10 root root 4096 Dec 13 14:54 hbasequeryweb
drwxr-xr-x  7 root root 4096 Oct  9 18:46 live_alarm_monitor
drwxr-xr-x  9 root root 4096 Oct 27 11:59 openrestywork
drwxr-xr-x  9 root root 4096 Dec 27 16:20 prepullvidcs
drwxr-xr-x  2 root root 4096 Nov 23 20:26 push_ffmpeg
drwxr-xr-x  3 root root 4096 Jan  4 18:46 testkafka
drwxr-xr-x  3 root root 4096 Nov 29 14:32 virtualenv
drwxr-xr-x  5 root root 4096 Oct 12 10:27 xiu_server_stat
```

单列显示某目录。

- `ls`

```
[root@hcdn-others-worker-dev100-bjlt home]# ls
ly  zabbix
```

显示当前目录下非影藏文件与目录

- `ls -a`

```
[root@hcdn-others-worker-dev100-bjlt home]# ls -a
.  ..  ly  zabbix
```

显示当前目录下包括影藏文件在内的所有文件列表

- `ls -l`

```
[root@hcdn-others-worker-dev100-bjlt home]# 
[root@hcdn-others-worker-dev100-bjlt home]# ls -l
total 8
drwxr-xr-x  15 root   root   4096 Jan  3 18:05 ly
drwxr-xr-x.  2 zabbix zabbix 4096 Dec 21 18:05 zabbix
```

输出长格式列表

- `ls -i -l `

```
[root@hcdn-others-worker-dev100-bjlt home]# ls -i -l
total 8
393222 drwxr-xr-x  15 root   root   4096 Jan  3 18:05 ly
393219 drwxr-xr-x.  2 zabbix zabbix 4096 Dec 21 18:05 zabbix
```

输出文件的inode信息

- `ls -m`

```
[root@hcdn-others-worker-dev100-bjlt home]# ls -m
ly, zabbix
```

水平输出文件列表

- `ls -t -l`

```
[root@hcdn-others-worker-dev100-bjlt ~]# ls -t -l
total 303312
drwxr-xr-x   2 root root       4096 Jan  4 17:54 testtar
drwxrwxr-x  10 root root       4096 Jan  4 15:21 librdkafka-0.11.1
drwxr-xr-x  11 root root       4096 Jan  4 14:26 pykafka
drwxr-xr-x   8 root root      12288 Jan  3 13:31 sarama
drwxr-xr-x  10 root root       4096 Jan  3 13:26 go_kafka_client
-rw-r--r--   1 root root     859238 Jan  3 13:01 librdkafka-0.11.1.tar.gz
drwxrwxr-x   7  500   500      4096 Jan  2 11:40 node-v8.9.3-linux-x64
drwxr-xr-x  10  502 games      4096 Jan  2 11:13 node-v8.9.3
-rw-r--r--   1 root root   31121503 Dec  8 23:23 node-v8.9.3.tar.gz
-rw-r--r--   1 root root   11395380 Dec  8 22:11 node-v8.9.3-linux-x64.tar.xz
-rw-r--r--   1 root root    1723533 Dec  5 01:02 redis-4.0.6.tar.gz
drwxrwxr-x   6 root root       4096 Dec  5 01:01 redis-4.0.6
drwxrwxr-x   6 1000  1000      4096 Oct 27 11:35 openresty-1.11.2.5
-rw-r--r--   1 root root    4183884 Oct 27 10:55 openresty-1.11.2.5.tar.gz
drwxr-xr-x   9 1001  1001      4096 Aug 30 20:05 nginx-1.12.1
drwxr-xr-x   9 1169  1169     12288 Aug 30 20:02 pcre-8.38
-rw-r--r--   1 root root    2053336 Aug 30 19:55 pcre-8.38.tar.gz
drwxr-xr-x  18 1000  1000      4096 Aug 28 15:15 Python-2.7.13
drwxr-xr-x  11 1000 ftp        4096 Aug 22 12:01 thrift-0.10.0
drwxr-xr-x  26 root root       4096 Aug 22 11:04 hbase-1.2.6
drwxr-xr-x   7  501 games      4096 Aug 11 09:26 Twisted-17.5.0
drwxr-xr-x  18  501   501      4096 Aug 11 09:13 Python-3.6.1
-rw-r--r--   1 root root     981093 Jul 11 23:45 nginx-1.12.1.tar.gz
-rw-r--r--   1 root root  194351104 Jul  7 15:25 long.flv
-rw-r--r--   1 root root    3340748 Jun 20  2017 thrift-0.10.0.tar.gz
-rw-r--r--   1 root root    2993816 Jun 11  2017 Twisted-17.5.0.tar.bz2
-rw-r--r--   1 root root   16054584 May 29  2017 hbase-1.2.6-src.tar.gz
-rw-r--r--   1 root root   22540566 Mar 21  2017 Python-3.6.1.tgz
-rw-r--r--   1 root root   17076672 Dec 18  2016 Python-2.7.13.tgz
-rw-r--r--   1 root root    1595408 Nov  7  2016 get-pip.py
-rw-------.  1 root root       1267 Mar 12  2014 anaconda-ks.cfg
-rw-r--r--   1 root root     197981 Apr 25  2009 pyOpenSSL-0.9.tar.gz
drwxr-xr-x   9 1125  1125      4096 Apr 25  2009 pyOpenSSL-0.9
```

最近修改的文件显示在最前面

- `ls -F -l`

```
[root@hcdn-others-worker-dev100-bjlt ~]# ls -F -l
total 303312
-rw-------.  1 root root       1267 Mar 12  2014 anaconda-ks.cfg
-rw-r--r--   1 root root    1595408 Nov  7  2016 get-pip.py
drwxr-xr-x  10 root root       4096 Jan  3 13:26 go_kafka_client/
drwxr-xr-x  26 root root       4096 Aug 22 11:04 hbase-1.2.6/
-rw-r--r--   1 root root   16054584 May 29  2017 hbase-1.2.6-src.tar.gz
drwxrwxr-x  10 root root       4096 Jan  4 15:21 librdkafka-0.11.1/
-rw-r--r--   1 root root     859238 Jan  3 13:01 librdkafka-0.11.1.tar.gz
-rw-r--r--   1 root root  194351104 Jul  7 15:25 long.flv
drwxr-xr-x   9 1001  1001      4096 Aug 30 20:05 nginx-1.12.1/
-rw-r--r--   1 root root     981093 Jul 11 23:45 nginx-1.12.1.tar.gz
drwxr-xr-x  10  502 games      4096 Jan  2 11:13 node-v8.9.3/
drwxrwxr-x   7  500   500      4096 Jan  2 11:40 node-v8.9.3-linux-x64/
-rw-r--r--   1 root root   11395380 Dec  8 22:11 node-v8.9.3-linux-x64.tar.xz
-rw-r--r--   1 root root   31121503 Dec  8 23:23 node-v8.9.3.tar.gz
drwxrwxr-x   6 1000  1000      4096 Oct 27 11:35 openresty-1.11.2.5/
-rw-r--r--   1 root root    4183884 Oct 27 10:55 openresty-1.11.2.5.tar.gz
drwxr-xr-x   9 1169  1169     12288 Aug 30 20:02 pcre-8.38/
-rw-r--r--   1 root root    2053336 Aug 30 19:55 pcre-8.38.tar.gz
drwxr-xr-x  11 root root       4096 Jan  4 14:26 pykafka/
drwxr-xr-x   9 1125  1125      4096 Apr 25  2009 pyOpenSSL-0.9/
-rw-r--r--   1 root root     197981 Apr 25  2009 pyOpenSSL-0.9.tar.gz
drwxr-xr-x  18 1000  1000      4096 Aug 28 15:15 Python-2.7.13/
-rw-r--r--   1 root root   17076672 Dec 18  2016 Python-2.7.13.tgz
drwxr-xr-x  18  501   501      4096 Aug 11 09:13 Python-3.6.1/
-rw-r--r--   1 root root   22540566 Mar 21  2017 Python-3.6.1.tgz
drwxrwxr-x   6 root root       4096 Dec  5 01:01 redis-4.0.6/
-rw-r--r--   1 root root    1723533 Dec  5 01:02 redis-4.0.6.tar.gz
drwxr-xr-x   8 root root      12288 Jan  3 13:31 sarama/
drwxr-xr-x   2 root root       4096 Jan  4 17:54 testtar/
drwxr-xr-x  11 1000 ftp        4096 Aug 22 12:01 thrift-0.10.0/
-rw-r--r--   1 root root    3340748 Jun 20  2017 thrift-0.10.0.tar.gz
drwxr-xr-x   7  501 games      4096 Aug 11 09:26 Twisted-17.5.0/
-rw-r--r--   1 root root    2993816 Jun 11  2017 Twisted-17.5.0.tar.bz2
```

安装特殊字符对文件进行分类

- `ls -l --color=auto | ls -l --color=never | ls -l --color=always`

```
[root@hcdn-others-worker-dev100-bjlt ~]# ls -l --color=auto
total 303312
-rw-------.  1 root root       1267 Mar 12  2014 anaconda-ks.cfg
-rw-r--r--   1 root root    1595408 Nov  7  2016 get-pip.py
drwxr-xr-x  10 root root       4096 Jan  3 13:26 go_kafka_client
drwxr-xr-x  26 root root       4096 Aug 22 11:04 hbase-1.2.6
-rw-r--r--   1 root root   16054584 May 29  2017 hbase-1.2.6-src.tar.gz
drwxrwxr-x  10 root root       4096 Jan  4 15:21 librdkafka-0.11.1
-rw-r--r--   1 root root     859238 Jan  3 13:01 librdkafka-0.11.1.tar.gz
-rw-r--r--   1 root root  194351104 Jul  7 15:25 long.flv
drwxr-xr-x   9 1001  1001      4096 Aug 30 20:05 nginx-1.12.1
-rw-r--r--   1 root root     981093 Jul 11 23:45 nginx-1.12.1.tar.gz
drwxr-xr-x  10  502 games      4096 Jan  2 11:13 node-v8.9.3
drwxrwxr-x   7  500   500      4096 Jan  2 11:40 node-v8.9.3-linux-x64
-rw-r--r--   1 root root   11395380 Dec  8 22:11 node-v8.9.3-linux-x64.tar.xz
-rw-r--r--   1 root root   31121503 Dec  8 23:23 node-v8.9.3.tar.gz
drwxrwxr-x   6 1000  1000      4096 Oct 27 11:35 openresty-1.11.2.5
-rw-r--r--   1 root root    4183884 Oct 27 10:55 openresty-1.11.2.5.tar.gz
drwxr-xr-x   9 1169  1169     12288 Aug 30 20:02 pcre-8.38
-rw-r--r--   1 root root    2053336 Aug 30 19:55 pcre-8.38.tar.gz
drwxr-xr-x  11 root root       4096 Jan  4 14:26 pykafka
drwxr-xr-x   9 1125  1125      4096 Apr 25  2009 pyOpenSSL-0.9
-rw-r--r--   1 root root     197981 Apr 25  2009 pyOpenSSL-0.9.tar.gz
drwxr-xr-x  18 1000  1000      4096 Aug 28 15:15 Python-2.7.13
-rw-r--r--   1 root root   17076672 Dec 18  2016 Python-2.7.13.tgz
drwxr-xr-x  18  501   501      4096 Aug 11 09:13 Python-3.6.1
-rw-r--r--   1 root root   22540566 Mar 21  2017 Python-3.6.1.tgz
drwxrwxr-x   6 root root       4096 Dec  5 01:01 redis-4.0.6
-rw-r--r--   1 root root    1723533 Dec  5 01:02 redis-4.0.6.tar.gz
drwxr-xr-x   8 root root      12288 Jan  3 13:31 sarama
drwxr-xr-x   2 root root       4096 Jan  4 17:54 testtar
drwxr-xr-x  11 1000 ftp        4096 Aug 22 12:01 thrift-0.10.0
-rw-r--r--   1 root root    3340748 Jun 20  2017 thrift-0.10.0.tar.gz
drwxr-xr-x   7  501 games      4096 Aug 11 09:26 Twisted-17.5.0
-rw-r--r--   1 root root    2993816 Jun 11  2017 Twisted-17.5.0.tar.bz2
[root@hcdn-others-worker-dev100-bjlt ~]# ls -l --color=never
total 303312
-rw-------.  1 root root       1267 Mar 12  2014 anaconda-ks.cfg
-rw-r--r--   1 root root    1595408 Nov  7  2016 get-pip.py
drwxr-xr-x  10 root root       4096 Jan  3 13:26 go_kafka_client
drwxr-xr-x  26 root root       4096 Aug 22 11:04 hbase-1.2.6
-rw-r--r--   1 root root   16054584 May 29  2017 hbase-1.2.6-src.tar.gz
drwxrwxr-x  10 root root       4096 Jan  4 15:21 librdkafka-0.11.1
-rw-r--r--   1 root root     859238 Jan  3 13:01 librdkafka-0.11.1.tar.gz
-rw-r--r--   1 root root  194351104 Jul  7 15:25 long.flv
drwxr-xr-x   9 1001  1001      4096 Aug 30 20:05 nginx-1.12.1
-rw-r--r--   1 root root     981093 Jul 11 23:45 nginx-1.12.1.tar.gz
drwxr-xr-x  10  502 games      4096 Jan  2 11:13 node-v8.9.3
drwxrwxr-x   7  500   500      4096 Jan  2 11:40 node-v8.9.3-linux-x64
-rw-r--r--   1 root root   11395380 Dec  8 22:11 node-v8.9.3-linux-x64.tar.xz
-rw-r--r--   1 root root   31121503 Dec  8 23:23 node-v8.9.3.tar.gz
drwxrwxr-x   6 1000  1000      4096 Oct 27 11:35 openresty-1.11.2.5
-rw-r--r--   1 root root    4183884 Oct 27 10:55 openresty-1.11.2.5.tar.gz
drwxr-xr-x   9 1169  1169     12288 Aug 30 20:02 pcre-8.38
-rw-r--r--   1 root root    2053336 Aug 30 19:55 pcre-8.38.tar.gz
drwxr-xr-x  11 root root       4096 Jan  4 14:26 pykafka
drwxr-xr-x   9 1125  1125      4096 Apr 25  2009 pyOpenSSL-0.9
-rw-r--r--   1 root root     197981 Apr 25  2009 pyOpenSSL-0.9.tar.gz
drwxr-xr-x  18 1000  1000      4096 Aug 28 15:15 Python-2.7.13
-rw-r--r--   1 root root   17076672 Dec 18  2016 Python-2.7.13.tgz
drwxr-xr-x  18  501   501      4096 Aug 11 09:13 Python-3.6.1
-rw-r--r--   1 root root   22540566 Mar 21  2017 Python-3.6.1.tgz
drwxrwxr-x   6 root root       4096 Dec  5 01:01 redis-4.0.6
-rw-r--r--   1 root root    1723533 Dec  5 01:02 redis-4.0.6.tar.gz
drwxr-xr-x   8 root root      12288 Jan  3 13:31 sarama
drwxr-xr-x   2 root root       4096 Jan  4 17:54 testtar
drwxr-xr-x  11 1000 ftp        4096 Aug 22 12:01 thrift-0.10.0
-rw-r--r--   1 root root    3340748 Jun 20  2017 thrift-0.10.0.tar.gz
drwxr-xr-x   7  501 games      4096 Aug 11 09:26 Twisted-17.5.0
-rw-r--r--   1 root root    2993816 Jun 11  2017 Twisted-17.5.0.tar.bz2
[root@hcdn-others-worker-dev100-bjlt ~]# ls -l --color=always
total 303312
-rw-------.  1 root root       1267 Mar 12  2014 anaconda-ks.cfg
-rw-r--r--   1 root root    1595408 Nov  7  2016 get-pip.py
drwxr-xr-x  10 root root       4096 Jan  3 13:26 go_kafka_client
drwxr-xr-x  26 root root       4096 Aug 22 11:04 hbase-1.2.6
-rw-r--r--   1 root root   16054584 May 29  2017 hbase-1.2.6-src.tar.gz
drwxrwxr-x  10 root root       4096 Jan  4 15:21 librdkafka-0.11.1
-rw-r--r--   1 root root     859238 Jan  3 13:01 librdkafka-0.11.1.tar.gz
-rw-r--r--   1 root root  194351104 Jul  7 15:25 long.flv
drwxr-xr-x   9 1001  1001      4096 Aug 30 20:05 nginx-1.12.1
-rw-r--r--   1 root root     981093 Jul 11 23:45 nginx-1.12.1.tar.gz
drwxr-xr-x  10  502 games      4096 Jan  2 11:13 node-v8.9.3
drwxrwxr-x   7  500   500      4096 Jan  2 11:40 node-v8.9.3-linux-x64
-rw-r--r--   1 root root   11395380 Dec  8 22:11 node-v8.9.3-linux-x64.tar.xz
-rw-r--r--   1 root root   31121503 Dec  8 23:23 node-v8.9.3.tar.gz
drwxrwxr-x   6 1000  1000      4096 Oct 27 11:35 openresty-1.11.2.5
-rw-r--r--   1 root root    4183884 Oct 27 10:55 openresty-1.11.2.5.tar.gz
drwxr-xr-x   9 1169  1169     12288 Aug 30 20:02 pcre-8.38
-rw-r--r--   1 root root    2053336 Aug 30 19:55 pcre-8.38.tar.gz
drwxr-xr-x  11 root root       4096 Jan  4 14:26 pykafka
drwxr-xr-x   9 1125  1125      4096 Apr 25  2009 pyOpenSSL-0.9
-rw-r--r--   1 root root     197981 Apr 25  2009 pyOpenSSL-0.9.tar.gz
drwxr-xr-x  18 1000  1000      4096 Aug 28 15:15 Python-2.7.13
-rw-r--r--   1 root root   17076672 Dec 18  2016 Python-2.7.13.tgz
drwxr-xr-x  18  501   501      4096 Aug 11 09:13 Python-3.6.1
-rw-r--r--   1 root root   22540566 Mar 21  2017 Python-3.6.1.tgz
drwxrwxr-x   6 root root       4096 Dec  5 01:01 redis-4.0.6
-rw-r--r--   1 root root    1723533 Dec  5 01:02 redis-4.0.6.tar.gz
drwxr-xr-x   8 root root      12288 Jan  3 13:31 sarama
drwxr-xr-x   2 root root       4096 Jan  4 17:54 testtar
drwxr-xr-x  11 1000 ftp        4096 Aug 22 12:01 thrift-0.10.0
-rw-r--r--   1 root root    3340748 Jun 20  2017 thrift-0.10.0.tar.gz
drwxr-xr-x   7  501 games      4096 Aug 11 09:26 Twisted-17.5.0
-rw-r--r--   1 root root    2993816 Jun 11  2017 Twisted-17.5.0.tar.bz2
```

单列列出文件并标记颜色

- `ls -l n*`

```
[root@hcdn-others-worker-dev100-bjlt ~]# ls -l n*
-rw-r--r--  1 root root    981093 Jul 11 23:45 nginx-1.12.1.tar.gz
-rw-r--r--  1 root root  11395380 Dec  8 22:11 node-v8.9.3-linux-x64.tar.xz
-rw-r--r--  1 root root  31121503 Dec  8 23:23 node-v8.9.3.tar.gz

nginx-1.12.1:
total 732
drwxr-xr-x 6 1001 1001   4096 Aug 30 20:00 auto
-rw-r--r-- 1 1001 1001 277349 Jul 11 21:24 CHANGES
-rw-r--r-- 1 1001 1001 422542 Jul 11 21:24 CHANGES.ru
drwxr-xr-x 2 1001 1001   4096 Aug 30 20:00 conf
-rwxr-xr-x 1 1001 1001   2481 Jul 11 21:24 configure
drwxr-xr-x 4 1001 1001   4096 Aug 30 20:00 contrib
drwxr-xr-x 2 1001 1001   4096 Aug 30 20:00 html
-rw-r--r-- 1 1001 1001   1397 Jul 11 21:24 LICENSE
-rw-r--r-- 1 root root    376 Aug 30 20:11 Makefile
drwxr-xr-x 2 1001 1001   4096 Aug 30 20:00 man
drwxr-xr-x 3 root root   4096 Aug 30 20:12 objs
-rw-r--r-- 1 1001 1001     49 Jul 11 21:24 README
drwxr-xr-x 9 1001 1001   4096 Aug 30 20:00 src

node-v8.9.3:
total 596
-rwxr-xr-x  1  502 games  1944 Dec  8 23:22 android-configure
-rw-r--r--  1  502 games 62552 Dec  8 23:22 AUTHORS
drwxr-xr-x 32  502 games  4096 Jan  2 11:13 benchmark
-rw-r--r--  1  502 games   263 Dec  8 23:22 BSDmakefile
-rw-r--r--  1  502 games 13117 Dec  8 23:22 BUILDING.md
-rw-r--r--  1  502 games 53576 Dec  8 23:22 CHANGELOG.md
-rw-r--r--  1  502 games   181 Dec  8 23:22 CODE_OF_CONDUCT.md
-rw-r--r--  1  502 games 30242 Dec  8 23:22 COLLABORATOR_GUIDE.md
-rw-r--r--  1  502 games 14659 Dec  8 23:22 common.gypi
-rw-r--r--  1 root root   2748 Jan  2 11:13 config.gypi
-rw-r--r--  1 root root    223 Jan  2 11:13 config.mk
-rwxr-xr-x  1  502 games 50245 Dec  8 23:22 configure
-rw-r--r--  1  502 games 37017 Dec  8 23:22 CONTRIBUTING.md
-rw-r--r--  1  502 games  3454 Dec  8 23:22 CPP_STYLE_GUIDE.md
drwxr-xr-x 13  502 games  4096 Jan  2 11:13 deps
drwxr-xr-x  6  502 games  4096 Jan  2 11:13 doc
-rw-r--r--  1  502 games  5832 Dec  8 23:22 GOVERNANCE.md
-rw-r--r--  1 root root  60153 Jan  2 11:13 icu_config.gypi
drwxr-xr-x  3  502 games  4096 Jan  2 11:13 lib
-rw-r--r--  1  502 games 59059 Dec  8 23:22 LICENSE
-rw-r--r--  1  502 games 37468 Dec  8 23:22 Makefile
-rw-r--r--  1  502 games 24881 Dec  8 23:23 node.gyp
-rw-r--r--  1  502 games 10585 Dec  8 23:23 node.gypi
drwxr-xr-x  5 root root   4096 Jan  2 11:14 out
-rw-r--r--  1  502 games 26525 Dec  8 23:22 README.md
drwxr-xr-x  4  502 games  4096 Jan  2 11:13 src
drwxr-xr-x 22  502 games  4096 Jan  2 11:13 test
drwxr-xr-x  9  502 games  4096 Jan  2 11:13 tools
-rw-r--r--  1  502 games 24869 Dec  8 23:23 vcbuild.bat

node-v8.9.3-linux-x64:
total 164
drwxrwxr-x 2  500  500  4096 Jan  2 12:14 bin
-rw-rw-r-- 1  500  500 53576 Dec  8 22:10 CHANGELOG.md
drwxr-xr-x 2 root root  4096 Jan  2 11:40 etc
drwxrwxr-x 3  500  500  4096 Dec  8 22:10 include
drwxrwxr-x 3  500  500  4096 Dec  8 22:10 lib
-rw-rw-r-- 1  500  500 59059 Dec  8 22:10 LICENSE
-rw-rw-r-- 1  500  500 26525 Dec  8 22:10 README.md
drwxrwxr-x 5  500  500  4096 Dec  8 22:10 share
```

单列列出以某个字符开头的文件和目录的详细内容



Read More: 

> [每天一个linux命令(1)：ls命令](http://www.cnblogs.com/peida/archive/2012/10/23/2734829.html) [Linux ls命令](http://www.runoob.com/linux/linux-comm-ls.html) [ls命令](http://man.linuxde.net/ls)  