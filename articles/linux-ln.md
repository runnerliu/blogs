## Linux 命令 - ln

Linux ln命令是一个非常重要命令，它的功能是为某一个文件在另外一个位置建立一个同步的链接。当我们需要在不同的目录，用到相同的文件时，我们不需要在每一个需要的目录下都放一个必须相同的文件，我们只要在某个固定的目录，放上该文件，然后在 其它的目录下用ln命令链接（link）它就可以，不必重复的占用磁盘空间。

#### 命令格式

```
ln [参数] [源文件或目录] [目标文件或目录]
```

#### 命令功能

Linux文件系统中，有所谓的链接（link），我们可以将其视为档案的别名，而链接又可分为两种：硬链接（hard link）与软链接（symbolic link），硬链接的意思是一个档案可以有多个名称，而软链接的方式则是产生一个特殊的档案，该档案的内容是指向另一个档案的位置。硬链接是存在同一个文件系统中，而软链接却可以跨越不同的文件系统。

不论是硬链接或软链接都不会将原本的档案复制一份，只会占用非常少量的磁碟空间。

软链接

- 软链接，以路径的形式存在。类似于Windows操作系统中的快捷方式
- 软链接可以 跨文件系统 ，硬链接不可以
- 软链接可以对一个不存在的文件名进行链接
- 软链接可以对目录进行链接

硬链接

- 硬链接，以文件副本的形式存在。但不占用实际空间
- 不允许给目录创建硬链接
- 硬链接只有在同一个文件系统中才能创建

注意

- ln命令会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化；
- ln的链接又分软链接和硬链接两种，软链接就是`ln –s 源文件 目标文件`，它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间，硬链接` ln 源文件 目标文件`，没有参数-s，它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持同步变化；
- ln指令用在链接文件或目录，如同时指定两个以上的文件或目录，且最后的目的地是一个已经存在的目录，则会把前面指定的所有文件或目录复制到该目录中。若同时指定多个文件或目录，且最后的目的地并非是一个已存在的目录，则会出现错误信息。

#### 命令参数

| 参数        | 说明                                    |
| --------- | ------------------------------------- |
| -b        | 删除，覆盖以前建立的链接                          |
| -d        | 允许超级用户制作目录的硬链接                        |
| -f        | 强制执行                                  |
| -i        | 交互模式，文件存在则提示用户是否覆盖                    |
| -n        | 把符号链接视为一般目录                           |
| -s        | 软链接(符号链接)                             |
| -v        | 显示详细的处理过程                             |
| -S        | "-S<字尾备份字符串> "或 "--suffix=<字尾备份字符串>"  |
| -V        | "-V<备份方式>"或"--version-control=<备份方式>" |
| --help    | 显示帮助信息                                |
| --version | 显示版本信息                                |

#### 使用示例

- ln -s cplog.log cplog-link.log

```
[root@hcdn-others-worker-dev100-bjlt testln]# ln -s cplog.log cplog-link.log 
[root@hcdn-others-worker-dev100-bjlt testln]# ll
total 4
lrwxrwxrwx 1 root root  9 Jan  3 20:25 cplog-link.log -> cplog.log
-rw-r--r-- 1 root root 28 Jan  3 20:00 cplog.log
```

为cplog.log文件创建软链接cplog-link.log，如果cplog.log丢失，其软链接cplog-link.log也将失效。

- ln cplog.log cplog-hard.log 

```
[root@hcdn-others-worker-dev100-bjlt testln]# ln cplog.log cplog-hard.log 
[root@hcdn-others-worker-dev100-bjlt testln]# ll
total 8
-rw-r--r-- 2 root root 28 Jan  3 20:00 cplog-hard.log
lrwxrwxrwx 1 root root  9 Jan  3 20:25 cplog-link.log -> cplog.log
-rw-r--r-- 2 root root 28 Jan  3 20:00 cplog.log
```

为cplog.log创建硬链接cplog-hard.log，cplog.log与cplog-hard.log的各项属性相同。

- ln cplog.log test/

```
[root@hcdn-others-worker-dev100-bjlt testln]# ln cplog.log test/
[root@hcdn-others-worker-dev100-bjlt testln]# ll
total 12
-rw-r--r-- 3 root root   28 Jan  3 20:00 cplog-hard.log
lrwxrwxrwx 1 root root    9 Jan  3 20:25 cplog-link.log -> cplog.log
-rw-r--r-- 3 root root   28 Jan  3 20:00 cplog.log
drwxr-xr-x 2 root root 4096 Jan  3 20:31 test
[root@hcdn-others-worker-dev100-bjlt testln]# vi cplog.log 
[root@hcdn-others-worker-dev100-bjlt testln]# ll
total 12
-rw-r--r-- 3 root root   72 Jan  3 20:31 cplog-hard.log
lrwxrwxrwx 1 root root    9 Jan  3 20:25 cplog-link.log -> cplog.log
-rw-r--r-- 3 root root   72 Jan  3 20:31 cplog.log
drwxr-xr-x 2 root root 4096 Jan  3 20:31 test
[root@hcdn-others-worker-dev100-bjlt testln]# ll test/
total 4
-rw-r--r-- 3 root root 72 Jan  3 20:31 cplog.log
```

将文件cplog.log创建硬链接到当前目录中已存在的目录，当修改./test/cplog.log文件时，会同步到源文件中。

- ln -sv /root/testln/test /root/testln/test2

```
[root@hcdn-others-worker-dev100-bjlt testln]# ll test/
total 0
[root@hcdn-others-worker-dev100-bjlt testln]# ll test2/
total 0
[root@hcdn-others-worker-dev100-bjlt testln]# ln -sv test test2
`test2/test' -> `test'
[root@hcdn-others-worker-dev100-bjlt testln]# ll test2
total 0
lrwxrwxrwx 1 root root 4 Jan  3 20:38 test -> test
[root@hcdn-others-worker-dev100-bjlt testln]# rm -rf test2/test 
[root@hcdn-others-worker-dev100-bjlt testln]# ll test2
total 0
[root@hcdn-others-worker-dev100-bjlt testln]# ln -sv /root/testln/test /root/testln/test2
`/root/testln/test2/test' -> `/root/testln/test'
[root@hcdn-others-worker-dev100-bjlt testln]# ll test2
total 0
lrwxrwxrwx 1 root root 17 Jan  3 20:39 test -> /root/testln/test
```

创建test目录到test2目录的软链接时，必须使用绝对路径，如果使用相对路径，则目的路径中的链接不断闪烁，表示链接失败。

目录链接只能是软链接，在链接目标目录中修改文件都会在源文件目录中同步变化。



Read More: 

> [每天一个linux命令（35）：ln 命令](http://www.cnblogs.com/peida/archive/2012/12/11/2812294.html) [Linux ln命令](http://www.runoob.com/linux/linux-comm-ln.html) 