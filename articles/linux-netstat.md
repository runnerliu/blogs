## Linux 命令 - netstat

### 命令格式

```
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```

### 命令功能

Linux netstat命令用于显示网络状态，一般用于检验本机各端口的网络连接情况。netstat是在内核中访问网络及相关信息的程序，它能提供TCP连接，TCP和UDP监听，进程内存管理的相关报告。

### 命令参数

| 参数                          | 说明                      |
| --------------------------- | ----------------------- |
| -a / -all                   | 显示所有连接中的Socket          |
| -A<网络类型> / --<网络类型>         | 列出该网络类型连接中的相关地址         |
| -c / --continuous           | 持续列出网络状态                |
| -C / --cache                | 显示路由器配置的快取信息            |
| -e / --extend               | 显示网络其他相关信息              |
| -F / --fib                  | 显示FIB                   |
| -g / --groups               | 显示多重广播功能群组组员名单          |
| -h / --help                 | 在线帮助                    |
| -i / --interfaces           | 显示网络界面信息表单              |
| -l / --listening            | 显示监控中的服务器的Socket        |
| -M / --masquerade           | 显示伪装的网络连线               |
| -n / --numeric              | 直接使用IP地址，而不通过域名服务器      |
| -N / --netlink / --symbolic | 显示网络硬件外围设备的符号连接名称       |
| -o / --timers               | 显示计时器                   |
| -p / --programs             | 显示正在使用Socket的程序识别码和程序名称 |
| -r / --route                | 显示路由表                   |
| -s / --statistice           | 显示网络工作信息统计表             |
| -t / --tcp                  | 显示TCP传输协议的连线状况          |
| -u / --udp                  | 显示UDP传输协议的连线状况          |
| -v / --verbose              | 显示指令执行过程                |
| -V / --version              | 显示版本信息                  |
| -w / --raw                  | 显示RAW传输协议的连线状况          |
| -x / --unix                 | 此参数的效果和指定"-A unix"参数相同  |
| --ip / --inet               | 此参数的效果和指定"-A inet"参数相同  |

### 使用示例

- netstat

```
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 hcdn-others-worker-de:62710 10.15.207.143:8433          ESTABLISHED 
tcp        0      0 hcdn-others-worker-de:30881 openlive-hbase-online:websm ESTABLISHED 
tcp        0      0 hcdn-others-worker-dev:9546 10.153.149.218:XmlIpcRegSvc ESTABLISHED 
tcp        0      0 hcdn-others-worker-de:53486 10.153.149.192:XmlIpcRegSvc ESTABLISHED 
tcp        0      0 hcdn-others-worker-de:38198 10.153.149.213:XmlIpcRegSvc ESTABLISHED 
tcp        0      0 hcdn-others-worker-de:24007 10.153.149.214:XmlIpcRegSvc ESTABLISHED 
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ]         DGRAM                    17227  @/org/freedesktop/hal/udev_event
unix  2      [ ]         DGRAM                    12672370 @/org/kernel/udev/udevd
unix  3      [ ]         STREAM     CONNECTED     253334334 
unix  3      [ ]         STREAM     CONNECTED     253334333 
```

`netstat` 命令的输出结果可以分为两个部分：Active Internet connections（有源TCP连接），其中"Recv-Q"和"Send-Q"指的是接收队列和发送队列，这些数字一般都应该是0，如果不是则表示软件包正在队列中堆积；Active UNIX domain sockets（有源Unix域套接口），和网络套接字一样，但只能用于本机通信，性能可以提高一倍。

Proto显示连接使用的协议，RefCnt显示连接到本套接口上的进程号，Type显示套接口的类型，State显示套接口当前的状态，Path表示连接到套接口的其它进程使用的路径名。

| 状态           | 说明                          |
| ------------ | --------------------------- |
| LISTEN       | 侦听来自远方的TCP端口的连接请求           |
| SYN-SENT     | 在发送连接请求后等待匹配的连接请求           |
| SYN-RECEIVED | 在收到和发送一个连接请求后等待对方对连接请求的确认   |
| ESTABLISHED  | 一个打开的连接                     |
| FIN-WAIT-1   | 等待远程TCP连接中断请求，或先前的连接中断请求的确认 |
| FIN-WAIT-2   | 从远程TCP等待连接中断请求              |
| CLOSE-WAIT   | 等待从本地用户发来的连接中断请求            |
| CLOSING      | 等待远程TCP对连接中断的确认             |
| LAST-ACK     | 等待原来的发向远程TCP的连接中断请求的确认      |
| TIME-WAIT    | 等待足够的时间以确保远程TCP接收到连接中断请求的确认 |
| CLOSED       | 没有任何连接状态                    |

- netstat -a

```
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 *:tproxy                    *:*                         LISTEN      
tcp        0      0 *:us-cli                    *:*                         LISTEN      
tcp        0      0 *:us-srv                    *:*                         LISTEN      
tcp        0      0 *:intu-ec-svcdisc           *:*                         LISTEN      
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ACC ]     STREAM     LISTENING     12688527 /var/run/abrt/abrt.socket
unix  2      [ ACC ]     STREAM     LISTENING     288268830 /tmp/supervisor.sock.3241
unix  2      [ ACC ]     STREAM     LISTENING     7341   @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     17201  @/var/run/hald/dbus-vA2CSxY4CU
unix  2      [ ACC ]     STREAM     LISTENING     17196  @/var/run/hald/dbus-5pcejJb6np
unix  2      [ ]         DGRAM                    17227  @/org/freedesktop/hal/udev_event
```

显示所有有效连接的列表，包括ESTABLISHED、LISTENING的连接。

- netstat -nu

```
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State 
```

显示当前UDP的连接情况

- netstat -apu

```
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address       Foreign Address             State       PID/Program name 
udp        0      0 *:947               *:*                                     1200/rpcbind     
udp        0      0 *:domain            *:*                                     9306/dnsmasq     
udp        0      0 *:983               *:*                                     1231/rpc.statd   
udp        0      0 *:sunrpc            *:*                                     1200/rpcbind     
udp        0      0 *:ipp               *:*                                     1141/portreserve 
udp        0      0 hcdn-others-worker-dev10:ntp *:*                            4247/ntpd       
udp        0      0 localhost:ntp       *:*                                     4247/ntpd       
udp        0      0 *:ntp               *:*                                     4247/ntpd       
udp        0      0 *:62630             *:*                                     1231/rpc.statd   
```

显示UDP端口号的使用情况

- netstat -i

```
Kernel Interface table
Iface       MTU Met    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
eth0       1500   0 2334710561      0      0      0 2493027803      0      0      0 BMRU
lo        65536   0 199796252      0      0      0 199796252      0      0      0 LRU
```

显示网卡列表

- netstat -g

```
IPv6/IPv4 Group Memberships
Interface       RefCnt Group
--------------- ------ ---------------------
lo              1      all-systems.mcast.net
eth0            1      all-systems.mcast.net
```

显示组播组的关系

- netstat -s

```
Ip:
    2402314477 total packets received
    0 forwarded
    0 incoming packets discarded
    2331780718 incoming packets delivered
    2692824123 requests sent out
Icmp:
    484304 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 159
        timeout in transit: 3
        echo requests: 83130
        echo replies: 400987
        timestamp request: 25
    505861 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 21437
        echo request: 401269
        echo replies: 83130
        timestamp replies: 25
IcmpMsg:
        InType0: 400987
        InType3: 159
        InType8: 83130
        InType11: 3
        InType13: 25
        OutType0: 83130
        OutType3: 21437
        OutType8: 401269
        OutType14: 25
Tcp:
    62593872 active connections openings
    177834 passive connection openings
    153086 failed connection attempts
    999 connection resets received
    89 connections established
    2250692557 segments received
    2608392230 segments send out
    3325843 segments retransmited
    132 bad segments received.
    610130 resets sent
Udp:
    80565656 packets received
    21451 packets to unknown port received.
    0 packet receive errors
    80600189 packets sent
UdpLite:
TcpExt:
    1100 invalid SYN cookies received
    361 resets received for embryonic SYN_RECV sockets
    466 packets pruned from receive queue because of socket buffer overrun
    61819153 TCP sockets finished time wait in fast timer
    2987 TCP sockets finished time wait in slow timer
    660268 delayed acks sent
    260 delayed acks further delayed because of locked socket
    Quick ack mode was activated 1051984 times
    25934905 packets directly queued to recvmsg prequeue.
    911172308 packets directly received from backlog
    3173278514 packets directly received from prequeue
    1186216864 packets header predicted
    3170682 packets header predicted and directly queued to user
    357381509 acknowledgments not containing data received
    411743100 predicted acknowledgments
    1093102 times recovered from packet loss due to SACK data
    Detected reordering 12642 times using FACK
    Detected reordering 14884 times using SACK
    Detected reordering 11 times using time stamp
    13885 congestion windows fully recovered
    252 congestion windows partially recovered using Hoe heuristic
    TCPDSACKUndo: 193430
    51009 congestion windows recovered after partial ack
    168112 TCP data loss events
    TCPLostRetransmit: 14829
    108524 timeouts after SACK recovery
    4274 timeouts in loss state
    2496500 fast retransmits
    158451 forward retransmits
    230933 retransmits in slow start
    280735 other TCP timeouts
    38730 sack retransmits failed
    23520 packets collapsed in receive queue due to low socket buffer
    1051963 DSACKs sent for old packets
    20 DSACKs sent for out of order packets
    365231 DSACKs received
    692 DSACKs for out of order packets received
    414 connections reset due to unexpected data
    896 connections reset due to early user close
    761 connections aborted due to timeout
    TCPDSACKIgnoredOld: 509
    TCPDSACKIgnoredNoUndo: 425
    TCPSackShiftFallback: 22772083
    TCPBacklogDrop: 2192
    TCPChallengeACK: 125
    TCPSYNChallenge: 125
    TCPFromZeroWindowAdv: 85
    TCPToZeroWindowAdv: 85
    TCPWantZeroWindowAdv: 163030
IpExt:
    InBcastPkts: 1232
    InOctets: 1540584137119
    OutOctets: 1474239597014
    InBcastOctets: 191368
```

显示网络统计信息

- netstat -l

```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 *:tproxy                    *:*                         LISTEN      
tcp        0      0 *:us-cli                    *:*                         LISTEN      
tcp        0      0 *:us-srv                    *:*                         LISTEN      
tcp        0      0 *:intu-ec-svcdisc           *:*                         LISTEN      
tcp        0      0 *:intu-ec-client            *:*                         LISTEN      
tcp        0      0 *:domain                    *:*                         LISTEN      
tcp        0      0 *:oa-system                 *:*                         LISTEN      
tcp        0      0 *:ssh                       *:*                         LISTEN      
tcp        0      0 *:8023                      *:*                         LISTEN      
tcp        0      0 localhost:smtp              *:*                         LISTEN      
tcp        0      0 *:12602                     *:*                         LISTEN      
tcp        0      0 hcdn-others-worker-de:27162 *:*                         LISTEN      
tcp        0      0 *:30050                     *:*                         LISTEN      
tcp        0      0 *:xmltec-xmlmail            *:*                         LISTEN      
tcp        0      0 hcdn-others-worker-de:21988 *:*                         LISTEN      
tcp        0      0 *:eforward                  *:*                         LISTEN      
tcp        0      0 *:svn                       *:*                         LISTEN      
tcp        0      0 *:16010                     *:*                         LISTEN      
tcp        0      0 *:6379                      *:*                         LISTEN      
tcp        0      0 *:23212                     *:*                         LISTEN      
tcp        0      0 hcdn-others-worker-de:28271 *:*                         LISTEN      
tcp        0      0 *:sunrpc                    *:*                         LISTEN      
tcp        0      0 *:http                      *:*                         LISTEN      
udp        0      0 *:947                       *:*                                     
udp        0      0 *:domain                    *:*                                     
udp        0      0 *:983                       *:*                                     
udp        0      0 *:sunrpc                    *:*                                     
udp        0      0 *:ipp                       *:*                                     
udp        0      0 hcdn-others-worker-dev10:ntp *:*                                     
udp        0      0 localhost:ntp               *:*                                     
udp        0      0 *:ntp                       *:*                                     
udp        0      0 *:62630                     *:*                                     
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ACC ]     STREAM     LISTENING     12688527 /var/run/abrt/abrt.socket
unix  2      [ ACC ]     STREAM     LISTENING     288268830 /tmp/supervisor.sock.3241
unix  2      [ ACC ]     STREAM     LISTENING     7341   @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     17201  @/var/run/hald/dbus-vA2CSxY4CU
unix  2      [ ACC ]     STREAM     LISTENING     17196  @/var/run/hald/dbus-5pcejJb6np
unix  2      [ ACC ]     STREAM     LISTENING     9429   /var/run/rpcbind.sock
unix  2      [ ACC ]     STREAM     LISTENING     9644   /var/run/cgred.socket
unix  2      [ ACC ]     STREAM     LISTENING     16960  /var/run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     17142  /var/run/acpid.socket
unix  2      [ ACC ]     STREAM     LISTENING     17754  /var/run/mcelog-client
unix  2      [ ACC ]     STREAM     LISTENING     18360  public/cleanup
unix  2      [ ACC ]     STREAM     LISTENING     18367  private/tlsmgr
unix  2      [ ACC ]     STREAM     LISTENING     18376  private/rewrite
unix  2      [ ACC ]     STREAM     LISTENING     18384  private/bounce
unix  2      [ ACC ]     STREAM     LISTENING     18388  private/defer
unix  2      [ ACC ]     STREAM     LISTENING     18392  private/trace
unix  2      [ ACC ]     STREAM     LISTENING     18397  private/verify
unix  2      [ ACC ]     STREAM     LISTENING     18401  public/flush
unix  2      [ ACC ]     STREAM     LISTENING     18405  private/proxymap
unix  2      [ ACC ]     STREAM     LISTENING     18409  private/proxywrite
unix  2      [ ACC ]     STREAM     LISTENING     18413  private/smtp
unix  2      [ ACC ]     STREAM     LISTENING     18417  private/relay
unix  2      [ ACC ]     STREAM     LISTENING     18421  public/showq
unix  2      [ ACC ]     STREAM     LISTENING     18425  private/error
unix  2      [ ACC ]     STREAM     LISTENING     18429  private/retry
unix  2      [ ACC ]     STREAM     LISTENING     18433  private/discard
unix  2      [ ACC ]     STREAM     LISTENING     18437  private/local
unix  2      [ ACC ]     STREAM     LISTENING     18441  private/virtual
unix  2      [ ACC ]     STREAM     LISTENING     18445  private/lmtp
unix  2      [ ACC ]     STREAM     LISTENING     18449  private/anvil
unix  2      [ ACC ]     STREAM     LISTENING     18453  private/scache
```

显示监听的套接口

- netstat -ap | grep python3

```
tcp        0      0 *:us-srv            *:*                         LISTEN      15484/python3   
tcp        0      0 *:xmltec-xmlmail    *:*                         LISTEN      14886/python3   
```

显示程序运行的端口

然后利用`ps -ef | grep '15484'` 命令可查找该端口运行的程序

```
root      6262   946  0 15:45 pts/0    00:00:00 grep 15484
root     15484     1  0  2017 ?        00:00:01 python3 /home/ly/atsbtsp/app.py
```



Read More：

> [每天一个linux命令（56）：netstat命令](http://www.cnblogs.com/peida/archive/2013/03/08/2949194.html) [Linux netstat命令](http://www.runoob.com/linux/linux-comm-netstat.html) 