## Linux 命令 - cp

Linux cp命令主要用于复制文件或目录，一般情况下，shell会设置一个别名，在命令行下复制文件时，如果目标文件已经存在，就会询问是否覆盖，不管你是否使用-i参数。但是如果是在shell脚本中执行cp时，没有-i参数时不会询问是否覆盖。这说明命令行和shell脚本的执行方式有些不同。 

### 命令格式

```
cp [options] source dest
cp [options] source... directory
```

### 命令功能

将源文件复制至目标文件，或将多个源文件复制至目标目录。

### 命令参数

| 参数      | 说明                                   |
| ------- | ------------------------------------ |
| -a      | 此选项通常在复制目录时使用，它保留链接、文件属性，并复制目录下的所有内容 |
| -b      | 为每个已存在的目标文件创建备份                      |
| -d      | 复制时保留链接。这里所说的链接相当于Windows系统中的快捷方式    |
| -f      | 覆盖已经存在的目标文件而不给出提示                    |
| -i      | 与-f选项相反，在覆盖目标文件之前给出提示                |
| -H      | 跟随源文件中的命令行符号链接                       |
| -l      | 不复制文件，只是生成链接文件                       |
| -L      | 总是跟随符号链接                             |
| -n      | 不要覆盖已存在的文件                           |
| -P      | 不跟随源文件中的符号链接                         |
| -p      | 除复制文件的内容外，还把修改时间和访问权限也复制到新文件中        |
| -R / -r | 若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件    |
| -s      | 复制时创建新的链接，类似于Windows系统中的快捷方式         |

### 使用示例

- cp cplog.log test1/

```
复制前：
[root@hcdn-others-worker-dev100-bjlt testcp]# ll
total 12
-rw-r--r-- 1 root root   28 Jan  3 20:00 cplog.log
drwxr-xr-x 2 root root 4096 Jan  3 20:00 test1
drwxr-xr-x 2 root root 4096 Jan  3 20:00 test2
[root@hcdn-others-worker-dev100-bjlt testcp]# ll test1/
total 0

复制后：
[root@hcdn-others-worker-dev100-bjlt testcp]# ll test1/
total 4
-rw-r--r-- 1 root root 28 Jan  3 20:01 cplog.log
```

在没有带-a参数时，两个文件的时间是不一样的。在带了-a参数时，两个文件的时间是一致的。  

- cp cplog.log test1/

```
[root@hcdn-others-worker-dev100-bjlt testcp]# cp cplog.log test1/
cp: overwrite `test1/cplog.log'? y
[root@hcdn-others-worker-dev100-bjlt testcp]# ll test1/
total 4
-rw-r--r-- 1 root root 28 Jan  3 20:04 cplog.log
```

在目标目录中存在同名文件时，会询问是否覆盖，这是因为cp是cp -i的别名。目标文件存在时，即使加了-f标志，也还会询问是否覆盖。

- cp -a test1 test2

```
目的目录存在：
[root@hcdn-others-worker-dev100-bjlt testcp]# ll
total 12
-rw-r--r-- 1 root root   28 Jan  3 20:00 cplog.log
drwxr-xr-x 2 root root 4096 Jan  3 20:01 test1
drwxr-xr-x 3 root root 4096 Jan  3 20:06 test2
[root@hcdn-others-worker-dev100-bjlt testcp]# ll test1
total 4
-rw-r--r-- 1 root root 28 Jan  3 20:04 cplog.log
[root@hcdn-others-worker-dev100-bjlt testcp]# ll test2
total 4
drwxr-xr-x 2 root root 4096 Jan  3 20:01 test1

目的目录不存在：
[root@hcdn-others-worker-dev100-bjlt testcp]# cp -a test1 test3
[root@hcdn-others-worker-dev100-bjlt testcp]# ll
total 16
-rw-r--r-- 1 root root   28 Jan  3 20:00 cplog.log
drwxr-xr-x 2 root root 4096 Jan  3 20:01 test1
drwxr-xr-x 3 root root 4096 Jan  3 20:06 test2
drwxr-xr-x 2 root root 4096 Jan  3 20:01 test3
[root@hcdn-others-worker-dev100-bjlt testcp]# ll test3/
total 4
-rw-r--r-- 1 root root 28 Jan  3 20:04 cplog.log
```

注意目标目录存在与否结果是不一样的。目标目录存在时，整个源目录被复制到目标目录里面。

- cp -s cplog.log cplog-1.log

```
[root@hcdn-others-worker-dev100-bjlt testcp]# cp -s cplog.log cplog-1.log 
[root@hcdn-others-worker-dev100-bjlt testcp]# ll
total 16
lrwxrwxrwx 1 root root    9 Jan  3 20:09 cplog-1.log -> cplog.log
-rw-r--r-- 1 root root   28 Jan  3 20:00 cplog.log
drwxr-xr-x 2 root root 4096 Jan  3 20:01 test1
drwxr-xr-x 3 root root 4096 Jan  3 20:06 test2
drwxr-xr-x 2 root root 4096 Jan  3 20:01 test3
```

cplog-1.log是由 -s 的参数造成的，建立的是一个快捷方式，所以会看到在文件的最右边，显示这个文件是连结到的目的地。



Read More: 

> [每天一个linux命令（8）：cp 命令](http://www.cnblogs.com/peida/archive/2012/10/29/2744185.html) [Linux cp命令](http://www.runoob.com/linux/linux-comm-cp.html) 