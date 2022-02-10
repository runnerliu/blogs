## Linux 命令 - ping

### 命令格式

```
ping [参数] [主机名称或IP地址]
```

### 命令功能

ping命令用来测试主机之间网络的连通性。执行ping指令会使用ICMP传输协议，发出要求回应的信息，若远端主机的网络功能没有问题，就会回应该信息，因而得知该主机运作正常。

ping 命令每秒发送一个数据报并且为每个接收到的响应打印一行输出。ping 命令计算信号往返时间和（信息）包丢失情况的统计信息，并且在完成之后显示一个简要总结。

### 命令参数

| 参数         | 说明                               |
| ---------- | -------------------------------- |
| -d         | 使用Socket的SO_DEBUG功能              |
| -c <完成次数>  | 设置完成要求回应的次数                      |
| -f         | 极限检测                             |
| -i <间隔秒数>  | 指定收发信息的间隔时间                      |
| -l <网络界面>  | 使用指定的网络界面送出数据包                   |
| -l <前置载入>  | 设置在送出要求信息之前，先行发出的数据包             |
| -n         | 只输出数值                            |
| -p         | 设置填满数据包的范本样式                     |
| -q         | 不显示指令执行过程，开头和结尾的相关信息除外           |
| -r         | 忽略普通的Routing Table，直接将数据包送到远端主机上 |
| -R         | 记录路由过程                           |
| -s <数据包大小> | 设置数据包的大小                         |
| -t <存活时间>  | 设置存活数值TTL的大小                     |
| -v         | 详细显示指令的执行过程                      |

### 使用示例

- ping 10.3.14.6

```
PING 10.3.14.62 (10.3.14.62) 56(84) bytes of data.
64 bytes from 10.3.14.62: icmp_seq=1 ttl=119 time=1.00 ms
64 bytes from 10.3.14.62: icmp_seq=2 ttl=119 time=0.921 ms
64 bytes from 10.3.14.62: icmp_seq=3 ttl=119 time=0.848 ms
64 bytes from 10.3.14.62: icmp_seq=4 ttl=119 time=0.862 ms
64 bytes from 10.3.14.62: icmp_seq=5 ttl=119 time=0.912 ms
^C
--- 10.3.14.62 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4990ms
rtt min/avg/max/mdev = 0.848/0.909/1.004/0.061 ms

需要手动Ctrl+c终止
```

ping通主机

- ping 10.3.14.6

```
PING 10.3.14.6 (10.3.14.6) 56(84) bytes of data.
From 10.13.41.63 icmp_seq=1 Destination Host Unreachable
From 10.13.41.634 icmp_seq=2 Destination Host Unreachable
From 10.13.41.63 icmp_seq=3 Destination Host Unreachable
From 10.13.41.63 icmp_seq=4 Destination Host Unreachable
From 10.13.41.63 icmp_seq=5 Destination Host Unreachable
From 10.13.41.63 icmp_seq=6 Destination Host Unreachable

--- 10.3.14.6 ping statistics ---
8 packets transmitted, 0 received, +6 errors, 100% packet loss, time 7005ms
, pipe 4
```

ping不通主机

- ping -c 10 10.3.14.62

```
PING 10.3.14.62 (10.3.14.62) 56(84) bytes of data.
64 bytes from 10.3.14.62: icmp_seq=1 ttl=119 time=0.914 ms
64 bytes from 10.3.14.62: icmp_seq=2 ttl=119 time=0.525 ms
64 bytes from 10.3.14.62: icmp_seq=3 ttl=119 time=1.09 ms
64 bytes from 10.3.14.62: icmp_seq=4 ttl=119 time=1.01 ms
64 bytes from 10.3.14.62: icmp_seq=5 ttl=119 time=0.956 ms
64 bytes from 10.3.14.62: icmp_seq=6 ttl=119 time=1.11 ms
64 bytes from 10.3.14.62: icmp_seq=7 ttl=119 time=0.870 ms
64 bytes from 10.3.14.62: icmp_seq=8 ttl=119 time=0.891 ms
64 bytes from 10.3.14.62: icmp_seq=9 ttl=119 time=0.922 ms
64 bytes from 10.3.14.62: icmp_seq=10 ttl=119 time=1.02 ms

--- 10.3.14.62 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9005ms
rtt min/avg/max/mdev = 0.525/0.932/1.115/0.158 ms
```

ping指定次数

- ping -c 10 -i 0.5 10.3.14.62

```
PING 10.3.14.62 (10.3.14.62) 56(84) bytes of data.
64 bytes from 10.3.14.62: icmp_seq=1 ttl=119 time=1.00 ms
64 bytes from 10.3.14.62: icmp_seq=2 ttl=119 time=0.889 ms
64 bytes from 10.3.14.62: icmp_seq=3 ttl=119 time=0.893 ms
64 bytes from 10.3.14.62: icmp_seq=4 ttl=119 time=0.865 ms
64 bytes from 10.3.14.62: icmp_seq=5 ttl=119 time=0.893 ms
64 bytes from 10.3.14.62: icmp_seq=6 ttl=119 time=0.868 ms
64 bytes from 10.3.14.62: icmp_seq=7 ttl=119 time=0.881 ms
64 bytes from 10.3.14.62: icmp_seq=8 ttl=119 time=0.885 ms
64 bytes from 10.3.14.62: icmp_seq=9 ttl=119 time=0.605 ms
64 bytes from 10.3.14.62: icmp_seq=10 ttl=119 time=0.569 ms

--- 10.3.14.62 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 4506ms
rtt min/avg/max/mdev = 0.569/0.835/1.003/0.130 ms
```

ping指定次数和时间间隔

- ping -c 5 www.zhihu.com

```
PING 6ej19t5k0le6q937.alicloudlayer.com (47.95.51.100) 56(84) bytes of data.
64 bytes from 47.95.51.100: icmp_seq=1 ttl=45 time=3.08 ms
64 bytes from 47.95.51.100: icmp_seq=2 ttl=45 time=3.02 ms
64 bytes from 47.95.51.100: icmp_seq=3 ttl=45 time=3.01 ms
64 bytes from 47.95.51.100: icmp_seq=4 ttl=45 time=3.05 ms
64 bytes from 47.95.51.100: icmp_seq=5 ttl=45 time=3.03 ms

--- 6ej19t5k0le6q937.alicloudlayer.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 3.012/3.042/3.088/0.074 ms
```

通过域名ping公网地址

- ping -i 3 -s 1024 -t 255 10.3.14.62

```
PING 10.3.14.62 (10.3.14.62) 1024(1052) bytes of data.
1032 bytes from 10.3.14.62: icmp_seq=1 ttl=119 time=1.18 ms
1032 bytes from 10.3.14.62: icmp_seq=2 ttl=119 time=1.21 ms
1032 bytes from 10.3.14.62: icmp_seq=3 ttl=119 time=1.22 ms
1032 bytes from 10.3.14.62: icmp_seq=4 ttl=119 time=1.19 ms
1032 bytes from 10.3.14.62: icmp_seq=5 ttl=119 time=1.20 ms
1032 bytes from 10.3.14.62: icmp_seq=6 ttl=119 time=1.23 ms
1032 bytes from 10.3.14.62: icmp_seq=7 ttl=119 time=1.22 ms
1032 bytes from 10.3.14.62: icmp_seq=8 ttl=119 time=1.22 ms
1032 bytes from 10.3.14.62: icmp_seq=9 ttl=119 time=1.21 ms
1032 bytes from 10.3.14.62: icmp_seq=10 ttl=119 time=1.21 ms
1032 bytes from 10.3.14.62: icmp_seq=11 ttl=119 time=1.19 ms
1032 bytes from 10.3.14.62: icmp_seq=12 ttl=119 time=1.28 ms
1032 bytes from 10.3.14.62: icmp_seq=13 ttl=119 time=1.20 ms
1032 bytes from 10.3.14.62: icmp_seq=14 ttl=119 time=1.23 ms
1032 bytes from 10.3.14.62: icmp_seq=15 ttl=119 time=1.21 ms
1032 bytes from 10.3.14.62: icmp_seq=16 ttl=119 time=1.21 ms
1032 bytes from 10.3.14.62: icmp_seq=17 ttl=119 time=1.18 ms
^C
--- 10.3.14.62 ping statistics ---
17 packets transmitted, 17 received, 0% packet loss, time 48798ms
rtt min/avg/max/mdev = 1.187/1.215/1.285/0.038 ms
```

ping多参数使用



Read More：

> [每天一个linux命令（54）：ping命令](http://www.cnblogs.com/peida/archive/2013/03/06/2945407.html) [Linux ping命令](http://www.runoob.com/linux/linux-comm-ping.html) 