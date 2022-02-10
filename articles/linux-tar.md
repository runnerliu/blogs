## Linux 命令 - tar

通过SSH访问服务器，难免会要用到压缩、解压缩、打包、解包等操作，这时候`tar`命令是必不可少的一个功能强大的工具。

tar命令可以为Linux的文件和目录创建档案。利用`tar`可以为某一特定文件创建档案（备份文件），也可以在档案中改变文件，或者向档案中加入新的文件。`tar`最初被用来在磁带上创建档案，现在，用户可以在任何设备上创建档案。

首先区分一下打包和压缩的概念：

- 打包是将一大堆文件或目录变成一个总的文件；
- 压缩是将一个大的文件通过一些压缩算法变成一个小文件。

为什么要区分这两个概念呢？这源于Linux中很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（`tar`命令），然后再用压缩程序进行压缩（`gzip`、`bzip2`命令）。

Linux下最常用的打包程序就是`tar`了，使用`tar`程序打出来的包我们常称为`tar`包，`tar`包文件的命令通常都是以`.tar`结尾的。生成`tar`包后，就可以用其它的程序来进行压缩。

### 命令格式

```
tar [参数] [文件]
```

### 命令功能

用来压缩和解压文件，`tar`本身不具有压缩功能，是调用压缩功能实现的压缩。

### 命令参数

| 参数             | 说明                            |
| -------------- | ----------------------------- |
| -A             | 新增文件到以存在的备份文件                 |
| -B             | 设置区块大小                        |
| -c             | 建立新的备份文件                      |
| -C <目录>        | 这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项 |
| -d             | 记录文件的差别                       |
| -r             | 添加文件到已经压缩的文件                  |
| -u             | 添加改变了和现有的文件到已经存在的压缩文件         |
| -x             | 从备份文件中还原文件                    |
| -t             | 列出备份文件的内容                     |
| -z             | 通过gzip指令处理备份文件                |
| -j             | 支持bzip2解压文件                   |
| -Z             | 通过compress指令处理备份文件            |
| -v             | 显示操作过程                        |
| -l             | 文件系统边界设置                      |
| -k             | 保留原有文件不覆盖                     |
| -m             | 保留文件不被覆盖                      |
| -w             | 确认压缩文件的正确性                    |
| -f <备份文件>      | 指定备份文件                        |
| -b <区块数目>      | 设置每笔记录的区块数目，每个区块大小为12Bytes    |
| -p             | 用原来的文件权限还原文件                  |
| -P             | 文件名使用绝对名称，不移除文件名称前的"/"号       |
| -N             | 只将较指定日期更新的文件保存到备份文件里          |
| --version      | 显示版本信息                        |
| --exclude <文件> | 排除某个文件                        |

### 常见的解压/压缩命令

`.tar`文件

```
解包：tar xvf FileName.tar
打包：tar cvf FileName.tar DirName
```

`.gz`文件

```
解压1：gunzip FileName.gz
解压2：gzip -d FileName.gz
压缩：gzip FileName
```

`.tar.gz`文件 和 `.tgz`文件

```
解压：tar zxvf FileName.tar.gz
压缩：tar zcvf FileName.tar.gz DirName
```

`.bz2`文件

```
解压1：bzip2 -d FileName.bz2
解压2：bunzip2 FileName.bz2
压缩： bzip2 -z FileName
```

`.tar.bz2`文件

```
解压：tar jxvf FileName.tar.bz2
压缩：tar jcvf FileName.tar.bz2 DirName
```

`.bz`文件

```
解压1：bzip2 -d FileName.bz
解压2：bunzip2 FileName.bz
压缩：未知
```

`.tar.bz`文件

```
解压：tar jxvf FileName.tar.bz
压缩：未知
```

`.Z`文件

```
解压：uncompress FileName.Z
压缩：compress FileName
```

`.tar.Z`文件

```
解压：tar Zxvf FileName.tar.Z
压缩：tar Zcvf FileName.tar.Z DirName
```

`.zip`文件

```
解压：unzip FileName.zip
压缩：zip FileName.zip DirName
```

`.rar`文件

```
解压：rar x FileName.rar
压缩：rar a FileName.rar DirName 
```

### 使用示例

- tar -cvf testlog.tar testlog.log | tar -zcvf testlog.tar.gz testlog.log | tar -jcvf testlog.tar.bz2 testlog.log

```
[root@hcdn-others-worker-dev100-bjlt testtar]# ll
total 4
-rw-r--r-- 1 root root 94 Jan  4 17:07 testlog.log
[root@hcdn-others-worker-dev100-bjlt testtar]# tar -cvf testlog.tar testlog.log 
testlog.log
[root@hcdn-others-worker-dev100-bjlt testtar]# tar -zcvf testlog.tar.gz testlog.log 
testlog.log
[root@hcdn-others-worker-dev100-bjlt testtar]# tar -jcvf testlog.tar.bz2 testlog.log 
testlog.log
[root@hcdn-others-worker-dev100-bjlt testtar]# ll
total 24
-rw-r--r-- 1 root root    94 Jan  4 17:07 testlog.log
-rw-r--r-- 1 root root 10240 Jan  4 17:08 testlog.tar
-rw-r--r-- 1 root root   128 Jan  4 17:08 testlog.tar.bz2
-rw-r--r-- 1 root root   125 Jan  4 17:08 testlog.tar.gz
```

将文件只打包不压缩、以gzip压缩、以bzip2压缩。

- tar -ztvf testlog.tar.gz

```
[root@hcdn-others-worker-dev100-bjlt testtar]# tar -ztvf testlog.tar.gz 
-rw-r--r-- root/root        94 2018-01-04 17:07 testlog.log
```

查看testlog.tar.gz中有哪些文件。

- tar -zxvf testlog.tar.gz

```
[root@hcdn-others-worker-dev100-bjlt testtar]# ll
total 24
drwxr-xr-x 2 root root  4096 Jan  4 17:17 testlog
-rw-r--r-- 1 root root 10240 Jan  4 17:08 testlog.tar
-rw-r--r-- 1 root root   128 Jan  4 17:08 testlog.tar.bz2
-rw-r--r-- 1 root root   125 Jan  4 17:08 testlog.tar.gz
[root@hcdn-others-worker-dev100-bjlt testtar]# tar -zxvf testlog.tar.gz
testlog.log
[root@hcdn-others-worker-dev100-bjlt testtar]# ll
total 28
drwxr-xr-x 2 root root  4096 Jan  4 17:17 testlog
-rw-r--r-- 1 root root    94 Jan  4 17:07 testlog.log
-rw-r--r-- 1 root root 10240 Jan  4 17:08 testlog.tar
-rw-r--r-- 1 root root   128 Jan  4 17:08 testlog.tar.bz2
-rw-r--r-- 1 root root   125 Jan  4 17:08 testlog.tar.gz
```

将testlog.tar.gz解压缩。

- tar -zcvf testlogs.tar.gz testlog1.log

```
[root@hcdn-others-worker-dev100-bjlt testtar]# tar -zcvf testlogs.tar.gz testlog1.log testlog2.log 
testlog1.log
testlog2.log
[root@hcdn-others-worker-dev100-bjlt testtar]# ll
total 36
drwxr-xr-x 2 root root  4096 Jan  4 17:17 testlog
-rw-r--r-- 1 root root    13 Jan  4 17:21 testlog1.log
-rw-r--r-- 1 root root    25 Jan  4 17:21 testlog2.log
-rw-r--r-- 1 root root   152 Jan  4 17:21 testlogs.tar.gz
-rw-r--r-- 1 root root 10240 Jan  4 17:08 testlog.tar
-rw-r--r-- 1 root root   128 Jan  4 17:08 testlog.tar.bz2
-rw-r--r-- 1 root root   125 Jan  4 17:08 testlog.tar.gz
[root@hcdn-others-worker-dev100-bjlt testtar]# tar -ztvf testlogs.tar.gz 
-rw-r--r-- root/root        13 2018-01-04 17:21 testlog1.log
-rw-r--r-- root/root        25 2018-01-04 17:21 testlog2.log
[root@hcdn-others-worker-dev100-bjlt testtar]# rm -rf testlog1.log 
[root@hcdn-others-worker-dev100-bjlt testtar]# ll
total 32
drwxr-xr-x 2 root root  4096 Jan  4 17:17 testlog
-rw-r--r-- 1 root root    25 Jan  4 17:21 testlog2.log
-rw-r--r-- 1 root root   152 Jan  4 17:21 testlogs.tar.gz
-rw-r--r-- 1 root root 10240 Jan  4 17:08 testlog.tar
-rw-r--r-- 1 root root   128 Jan  4 17:08 testlog.tar.bz2
-rw-r--r-- 1 root root   125 Jan  4 17:08 testlog.tar.gz
[root@hcdn-others-worker-dev100-bjlt testtar]# tar -zxvf testlogs.tar.gz testlog1.log
testlog1.log
[root@hcdn-others-worker-dev100-bjlt testtar]# ll
total 36
drwxr-xr-x 2 root root  4096 Jan  4 17:17 testlog
-rw-r--r-- 1 root root    13 Jan  4 17:21 testlog1.log
-rw-r--r-- 1 root root    25 Jan  4 17:21 testlog2.log
-rw-r--r-- 1 root root   152 Jan  4 17:21 testlogs.tar.gz
-rw-r--r-- 1 root root 10240 Jan  4 17:08 testlog.tar
-rw-r--r-- 1 root root   128 Jan  4 17:08 testlog.tar.bz2
-rw-r--r-- 1 root root   125 Jan  4 17:08 testlog.tar.gz
```

从压缩包中解压出指定文件。

- tar -N "2018/01/03" -zcvf testlog.tar.gz test: 指定文件夹中的文件比某个日期新的才备份
- tar --exclude testlog/testlog1.log -zcvf testlogs.tar.gz testlog/*: 排除文件夹中的某个文件



Read More：

> [每天一个linux命令（28）：tar命令](http://www.cnblogs.com/peida/archive/2012/11/30/2795656.html) [Linux tar命令](http://www.runoob.com/linux/linux-comm-tar.html)  [tar](http://wangchujiang.com/linux-command/c/tar.html) 