## Linux 命令 - cd

Linux cd命令用于切换当前工作目录至目标目录。其中目标目录表示法可为绝对路径或相对路径，若目录名称省略，则变换至使用者的 `home` 目录。

另外，"~" 也表示为 `home` 目录的意思，"." 则是表示当前所在的目录，".." 则表示目前目录位置的上一层目录。

### 命令格式

```
cd [dir](dir表示要切换的目录)
```

### 命令功能

切换当前目录至dir。

### 命令参数

暂无

### 使用示例

- `cd / | cd //`

```
[root@hcdn-others-worker-dev100-bjlt local]# pwd
/root/usr/local
[root@hcdn-others-worker-dev100-bjlt local]# cd /
[root@hcdn-others-worker-dev100-bjlt /]# pwd
/
```

进入系统根目录

- `cd ..`

```
[root@hcdn-others-worker-dev100-bjlt local]# pwd
/root/usr/local
[root@hcdn-others-worker-dev100-bjlt local]# cd ..
[root@hcdn-others-worker-dev100-bjlt usr]# pwd
/root/usr
```

切换到父目录

- `cd -`

```
[root@hcdn-others-worker-dev100-bjlt local]# pwd
/root/usr/local
[root@hcdn-others-worker-dev100-bjlt local]# cd -
/root/usr
[root@hcdn-others-worker-dev100-bjlt usr]# cd
[root@hcdn-others-worker-dev100-bjlt ~]# cd -
/root/usr
```

切换到上次使用的目录

- `cd`

```
[root@hcdn-others-worker-dev100-bjlt local]# pwd
/root/usr/local
[root@hcdn-others-worker-dev100-bjlt local]# cd
[root@hcdn-others-worker-dev100-bjlt ~]# pwd
/root
```

切换到当前用户主目录

- `cd ~`

```
[root@hcdn-others-worker-dev100-bjlt ~]# cd /root/usr/local/
[root@hcdn-others-worker-dev100-bjlt local]# cd ~
[root@hcdn-others-worker-dev100-bjlt ~]# pwd
/root
```

切换到当前用户主目录

- `cd /root/usr/local/`

```
[root@hcdn-others-worker-dev100-bjlt ~]# cd /root/usr/local/
[root@hcdn-others-worker-dev100-bjlt local]# pwd
/root/usr/local
```

切换到指定目录

- `cd !$`

```
[root@hcdn-others-worker-dev100-bjlt ~]# cd /root/usr/local/
[root@hcdn-others-worker-dev100-bjlt local]# cd -
/root
[root@hcdn-others-worker-dev100-bjlt ~]# cd !$
cd -
/root/usr/local
```

将上一命令得参数作为 `cd` 参数使用



Read More: 

> [每天一个linux命令(2)：cd命令](http://www.cnblogs.com/peida/archive/2012/10/24/2736501.html) [Linux cd命令](http://www.runoob.com/linux/linux-comm-cd.html) 