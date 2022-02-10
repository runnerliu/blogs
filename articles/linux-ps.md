## Linux 命令 - ps

Linux中的ps命令是Process Status的缩写。用来列出系统中当前运行的那些进程。ps命令列出的是当前那些进程的快照，就是执行ps命令的那个时刻的那些进程，如果想要动态的显示进程信息，就可以使用top命令。

要对进程进行监测和控制，首先必须要了解当前进程的情况，也就是需要查看当前进程，ps 命令是最基本同时也是非常强大的进程查看命令。使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等。总之大部分信息都是可以通过执行该命令得到的。

### Linux上的进程状态

- 运行（正在运行或在运行队列中等待）
- 中断（休眠中，受阻，在等待某个条件的形成或接受到信号）
- 不可中断（收到信号不唤醒和不可运行，进程必须等待直到有中断发生）
- 僵死（进程已终止，但进程描述符存在，直到父进程调用wait4()系统调用后释放）
- 停止（进程收到SIGSTOP、SIGSTP、SIGTIN、SIGTOU信号后停止运行运行）

### ps命令标识进程状态码

- R: 运行
- S: 中断
- D: 不可中断
- Z: 僵死
- T: 停止

### 命令格式

```
ps [参数]
```

### 命令功能

ps命令用来显示当前进程的状态。

### 命令参数

| 参数            | 说明             |
| ------------- | -------------- |
| a             | 显示所有进程         |
| -a            | 显示同一终端下的所有程序   |
| -A            | 显示所有进程         |
| c             | 显示进程的真实名称      |
| -N            | 反向选择           |
| -e            | 等于"-A"         |
| e             | 显示环境变量         |
| f             | 显示程序间的关系       |
| -H            | 显示树状结构         |
| r             | 显示当前终端的进程      |
| T             | 显示当前终端的所有程序    |
| u             | 指定用户的所有进程      |
| -au           | 显示较详细的资讯       |
| -aux          | 显示所有包含其他使用者的行程 |
| -C <命令>d      | 列出指定命令的状况      |
| --lines <行数>  | 每页显示的行数        |
| --width <字符数> | 每页显示的字符数       |
| --help        | 显示帮助信息         |
| --version     | 显示版本显示         |

### 使用示例

- ps -A

```
PID TTY          TIME CMD
    1 ?        00:00:16 init
    2 ?        00:00:00 kthreadd
    3 ?        00:00:21 migration/0
    4 ?        00:00:36 ksoftirqd/0
    5 ?        00:00:00 stopper/0
    6 ?        00:00:08 watchdog/0
    7 ?        00:00:21 migration/1
    8 ?        00:00:00 stopper/1
    9 ?        00:00:35 ksoftirqd/1
   10 ?        00:00:07 watchdog/1
   11 ?        00:00:21 migration/2
   12 ?        00:00:00 stopper/2
   13 ?        00:00:34 ksoftirqd/2
```

显示所有进程

- ps -u root

```
PID TTY          TIME CMD
    1 ?        00:00:16 init
    2 ?        00:00:00 kthreadd
    3 ?        00:00:21 migration/0
    4 ?        00:00:36 ksoftirqd/0
    5 ?        00:00:00 stopper/0
    6 ?        00:00:08 watchdog/0
    7 ?        00:00:21 migration/1
    8 ?        00:00:00 stopper/1
    9 ?        00:00:35 ksoftirqd/1
   10 ?        00:00:07 watchdog/1
```

显示特定用户的进程信息

- ps -ef

```
UID        PID  PPID  C STIME TTY          TIME CMD
root     12426 10831  0 16:47 pts/0    00:00:00 ps -ef
root     14886     1  0  2017 ?        00:00:01 python3 /home/ly/prepullvidcs/app.py
root     15484     1  0  2017 ?        00:00:01 python3 /home/ly/atsbtsp/app.py
root     17080     1  0  2017 ?        00:00:00 bash /usr/local/hbase-1.2.6/bin/hbase-daemon.sh --config /usr/local/hbase-1.2.6/bin/../conf foreground_start master
root     17094 17080  0  2017 ?        04:35:00 /usr/lib/java-1.8/jdk1.8.0_144/bin/java -Dproc_master -XX:OnOutOfMemoryError=kill -9 %p -XX:+UseConcMarkSweepGC -XX:PermSize=128m -XX:MaxPermSize=128m -Dhbase.log.dir=/usr/local/hbase-1.2.6/
root     17890     1  0  2017 ?        00:05:22 /usr/local/zabbix/share/zabbix/externalscripts/zabbix-agentd-updater
root     17973 18967  0  2017 ?        00:00:00 /sbin/udevd -d
root     18634     1  0  2017 ?        00:18:54 python27 /home/ly/datawritehbase/tasks/live_stream_ugcppcpgc/stream_list_old.py
```

显示所有进程信息（包括命令行）

- ps -ef | grep python

```
root      3244     1  0  2017 ?        00:02:29 /usr/bin/python27 /usr/local/python2.7/bin/supervisord -c /etc/supervisor/supervisord.conf
root      3245  3244  0  2017 ?        00:00:01 python27 /home/ly/analysisweb/app.py --port=8021
root      3246  3244  0  2017 ?        00:00:01 python27 /home/ly/analysisweb/app.py --port=8020
root      3247  3244  0  2017 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8023
root      3248  3244  0  2017 ?        00:00:00 python27 /home/ly/analysisweb/app.py --port=8022
root      8472     1  0  2017 ?        00:01:20 python3 monitor.py tasks/live_stream_length_task.py
850      12515  9226  1 16:48 ?        00:00:00 /opt/cloud-agent/agent/plugin/env/bin/python plugin/python/sys/basic/60_ifstat.py
root     12567 10831  0 16:48 pts/0    00:00:00 grep python
root     14886     1  0  2017 ?        00:00:01 python3 /home/ly/prepullvidcs/app.py
root     15484     1  0  2017 ?        00:00:01 python3 /home/ly/atsbtsp/app.py
root     18634     1  0  2017 ?        00:18:54 python27 /home/ly/datawritehbase/tasks/live_stream_ugcppcpgc/stream_list_old.py
root     18635     1 99  2017 ?        28-05:56:25 python27 /home/ly/datawritehbase/tasks/multicdn_kafka_hbase/mcdn_main.py
root     18636     1 99  2017 ?        28-20:46:31 python27 /home/ly/datawritehbase/tasks/nginx_kafka_hbase/nginx_main.py
root     18638     1  3  2017 ?        20:43:58 python27 /home/ly/datawritehbase/tasks/stream_hbase/stream_main_old.py
root     20253     1  0  2017 ?        00:01:00 python27 /home/ly/hbasequeryweb/app.py
root     30087     1  1  2017 ?        11:27:44 python27 /home/ly/datawritehbase/tasks/rtmp_kafka_hbase/rtmp_main.py
```

查找特定进程

- ps aux

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  19356  1544 ?        Ss    2017   0:16 /sbin/init
root         2  0.0  0.0      0     0 ?        S     2017   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S     2017   0:21 [migration/0]
root         4  0.0  0.0      0     0 ?        S     2017   0:36 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S     2017   0:00 [stopper/0]
root         6  0.0  0.0      0     0 ?        S     2017   0:08 [watchdog/0]
root         7  0.0  0.0      0     0 ?        S     2017   0:21 [migration/1]
root         8  0.0  0.0      0     0 ?        S     2017   0:00 [stopper/1]
root         9  0.0  0.0      0     0 ?        S     2017   0:35 [ksoftirqd/1]
root        10  0.0  0.0      0     0 ?        S     2017   0:07 [watchdog/1]
root        11  0.0  0.0      0     0 ?        S     2017   0:21 [migration/2]
......
root      1392  0.0  0.0      0     0 ?        S     2017   1:29 [flush-252:0]
root      3244  0.0  0.0 209168 14424 ?        Ss    2017   2:29 /usr/bin/python27 /usr/local/python2.7/bin/supervisord -c /etc/supervisor/supervisord.conf
root      3245  0.0  0.2 248368 37396 ?        S     2017   0:01 python27 /home/ly/analysisweb/app.py --port=8021
root      3246  0.0  0.2 253408 42800 ?        S     2017   0:01 python27 /home/ly/analysisweb/app.py --port=8020
root      3247  0.0  0.2 243556 32756 ?        S     2017   0:00 python27 /home/ly/analysisweb/app.py --port=8023
root      3248  0.0  0.2 245124 34528 ?        S     2017   0:00 python27 /home/ly/analysisweb/app.py --port=8022
dbus      3857  0.0  0.0  21540  1364 ?        Ss    2017   0:00 dbus-daemon --system
root      3886  0.0  0.0   4076   648 ?        Ss    2017   0:00 /usr/sbin/acpid
68        3895  0.0  0.0  25068  3848 ?        Ss    2017   0:37 hald
root      3896  0.0  0.0  18104  1132 ?        S     2017   0:00 hald-runner
root      3924  0.0  0.0  20220  1080 ?        S     2017   0:00 hald-addon-input: Listening on /dev/input/event2 /dev/input/event0
68        3934  0.0  0.0  17804  1036 ?        S     2017   0:00 hald-addon-acpi: listening on acpid socket /var/run/acpid.socket
root      3981  0.0  0.0   6244   292 ?        Ss    2017   0:00 /usr/sbin/mcelog --daemon
root      4093  0.0  0.0  78728  3312 ?        Ss    2017   0:30 /usr/libexec/postfix/master
postfix   4108  0.0  0.0  78980  3444 ?        S     2017   0:04 qmgr -l -t fifo -u
root      4159  0.0  0.0  21452   476 ?        Ss    2017   0:00 /usr/sbin/atd
......
root     10826  0.0  0.0 101072  4024 ?        Ss   16:30   0:00 sshd: root@pts/0 
root     10831  0.0  0.0 109096  2556 pts/0    Ss   16:30   0:00 -bash
root     12865  0.0  0.0 110240  1136 pts/0    R+   16:51   0:00 ps aux
root     14886  0.0  0.2 533372 40212 ?        Sl    2017   0:01 python3 /home/ly/prepullvidcs/app.py
root     15484  0.0  0.1 233608 30092 ?        S     2017   0:01 python3 /home/ly/atsbtsp/app.py
root     17080  0.0  0.0 106100  1380 ?        S     2017   0:00 bash /usr/local/hbase-1.2.6/bin/hbase-daemon.sh --config /usr/local/hbase-1.2.6/bin/../conf foreground_start master
root     17094  0.1  9.1 6203172 1486980 ?     Sl    2017 275:00 /usr/lib/java-1.8/jdk1.8.0_144/bin/java -Dproc_master -XX:OnOutOfMemoryError=kill -9 %p -XX:+UseConcMarkSweepGC -XX:PermSize=128m -XX:MaxPermSize=128m -Dhbase.log.dir=/usr/l
root     17890  0.0  0.0 587052 10368 ?        Sl    2017   5:22 /usr/local/zabbix/share/zabbix/externalscripts/zabbix-agentd-updater
root     17973  0.0  0.0  11540   676 ?        S<    2017   0:00 /sbin/udevd -d
......
root     25290  0.0  0.0 175824  1008 ?        Ss    2017   0:00 svnserve -dr /var/svndata/
root     25793  0.0  0.0  76792  1384 ?        S    03:28   0:00 /usr/local/zabbix/sbin/zabbix_agentd_ops 
root     25794  0.0  0.0  76792  1472 ?        S    03:28   0:07 /usr/local/zabbix/sbin/zabbix_agentd_ops: collector [idle 1 sec]
root     25795  0.0  0.0  76792  1040 ?        S    03:28   0:00 /usr/local/zabbix/sbin/zabbix_agentd_ops: listener #1 [waiting for connection]
root     25796  0.0  0.0  78984  2008 ?        S    03:28   0:06 /usr/local/zabbix/sbin/zabbix_agentd_ops: active checks #1 [idle 1 sec]
root     25797  0.0  0.0  78984  1996 ?        S    03:28   0:06 /usr/local/zabbix/sbin/zabbix_agentd_ops: active checks #2 [idle 1 sec]
root     26648  0.0  0.0 125472  7688 ?        Ssl   2017   5:33 ./redis-server *:6379      
root     30087  1.7  0.6 7316724 106248 ?      Sl    2017 687:47 python27 /home/ly/datawritehbase/tasks/rtmp_kafka_hbase/rtmp_main.py
```

列出目前所有正在内存中的进程

| 参数      | 说明                                       |
| ------- | ---------------------------------------- |
| USER    | 进程所属账号                                   |
| PID     | 进程id号                                    |
| %CPU    | 进程所占CPU资源百分比                             |
| %MEM    | 进程所占物理内存百分比                              |
| VSZ     | 进程所占虚拟内存大小                               |
| RSS     | 进程所占固定内存大小                               |
| TTY     | 进程在哪个终端机上运行。若与终端机无关，则显示 ？；tty1-tty6 是本机上面的登入者程序；若为 pts/0 等等的，则表示为由网络连接进主机的程序 |
| STAT    | 进程当前状态                                   |
| START   | 进程被触发启动的时间                               |
| TIME    | 进程实际使用CPU的时间                             |
| COMMAND | 进程的启动指令                                  |

- ps aux | egrep '(cron|syslog)'

```
root      9278  0.0  0.0 117256  1276 ?        Ss   11:10   0:00 crond
root     13816  0.0  0.0 101016   844 pts/0    S+   17:00   0:00 egrep (cron|syslog)
root     22016  0.0  0.0 159736  5856 ?        Sl    2017   2:06 /sbin/rsyslogd -i /var/run/syslogd.pid -c 5
```

显示与 cron、syslog服务有关进程的PID号

- ps -aux | more : 分页查看
- ps -aux > ps001.txt : 输出到文件中
- ps -o pid,ppid,pgrp,session,tpgid,comm : 输出指定字段



Read More：

> [每天一个linux命令（41）：ps命令](http://www.cnblogs.com/peida/archive/2012/12/19/2824418.html) [Linux ps命令](http://www.runoob.com/linux/linux-comm-ps.html)