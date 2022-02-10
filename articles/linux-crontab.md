## Linux 命令 - crontab

crond是linux下用来周期性的执行某种任务或等待处理某些事件的一个守护进程，与windows下的计划任务类似，当安装完成操作系统后，默认会安装此服务工具，并且会自动启动crond进程，crond进程每分钟会定期检查是否有要执行的任务，如果有要执行的任务，则自动执行该任务。

### Linux下的任务调度

Linux下的任务调度分为两类，系统任务调度和用户任务调度。

#### 系统任务调度

系统周期性所要执行的工作，比如写缓存数据到硬盘、日志清理等。在/etc目录下有一个crontab文件，这个就是系统任务调度的配置文件。

/etc/crontab文件包括下面几行：

```
[root@localhost ~]# cat /etc/crontab 
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=""HOME=/
# run-parts
51 * * * * root run-parts /etc/cron.hourly
24 7 * * * root run-parts /etc/cron.daily
22 4 * * 0 root run-parts /etc/cron.weekly
42 4 1 * * root run-parts /etc/cron.monthly
[root@localhost ~]#
```

前四行是用来配置crond任务运行的环境变量，第一行SHELL变量指定了系统要使用哪个shell，这里是bash，第二行PATH变量指定了系统执行命令的路径，第三行MAILTO变量指定了crond的任务执行信息将通过电子邮件发送给root用户，如果MAILTO变量的值为空，则表示不发送任务执行信息给用户，第四行的HOME变量指定了在执行命令或者脚本时使用的主目录。

#### 用户任务调度

用户定期要执行的工作，比如用户数据备份、定时邮件提醒等。用户可以使用 crontab 工具来定制自己的计划任务。所有用户定义的crontab 文件都被保存在 /var/spool/cron 目录中。其文件名与用户名一致。

### 使用者权限文件

| 文件               | 说明                        |
| ---------------- | ------------------------- |
| /etc/cron.deny   | 该文件中所列用户不允许使用crontab命令    |
| /etc/cron.allow  | 该文件中所列用户允许使用crontab命令     |
| /var/spool/cron/ | 所有用户crontab文件存放的目录，以用户名命名 |

### crontab文件的含义

用户所建立的crontab文件中，每一行都代表一项任务，每行的每个字段代表一项设置，它的格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段，格式如下：

```
minute   hour   day   month   week   command
```

| 参数      | 说明                               |
| ------- | -------------------------------- |
| minute  | 表示分钟，可以是从0到59之间的任何整数             |
| hour    | 表示小时，可以是从0到23之间的任何整数             |
| day     | 表示日期，可以是从1到31之间的任何整数             |
| month   | 表示月份，可以是从1到12之间的任何整数             |
| week    | 表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日 |
| command | 要执行的命令，可以是系统命令，也可以是自己编写的脚本文件     |

![2017-4-2165636](../images/2017-4-2 165636.png)

在以上各个字段中，还可以使用以下特殊字符：

星号(\*)：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作；

逗号(,)：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”；

中杠(-)：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”；

正斜线(/)：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如\*/10，如果用在minute字段，表示每十分钟执行一次。

### crond服务

安装crontab：

```
yum install crontabs
```

服务操作说明：

```
/sbin/service crond start    //启动服务
/sbin/service crond stop     //关闭服务
/sbin/service crond restart  //重启服务
/sbin/service crond reload   //重新载入配置
```

查看crontab服务状态：

```
service crond status
```

手动启动crontab服务：

```
service crond start
```

查看crontab服务是否已设置为开机启动，执行命令：

```
ntsysv
```

加入开机自动启动：

```
chkconfig –level 35 crond on
```

### crontab命令详解

#### 命令格式

```
crontab [-u user] file
crontab [-u user] [ -e | -l | -r ]
```

#### 命令功能

通过crontab 命令，我们可以在固定的间隔时间执行指定的系统指令或 shell script脚本。时间间隔的单位可以是分钟、小时、日、月、周及以上的任意组合。这个命令非常设合周期性的日志分析或数据备份等工作。

#### 命令参数

| 参数      | 说明                                       |
| ------- | ---------------------------------------- |
| -u user | 用来设定某个用户的crontab服务，例如，“-u ixdba”表示设定ixdba用户的crontab服务，此参数一般有root用户来运行 |
| file    | file是命令文件的名字,表示将file做为crontab的任务列表文件并载入crontab。如果在命令行中没有指定这个文件，crontab命令将接受标准输入（键盘）上键入的命令，并将它们载入crontab |
| -e      | 编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件 |
| -l      | 显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容 |
| -r      | 从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件 |
| -i      | 在删除用户的crontab文件时给确认提示                    |

#### 使用示例

| 示例                          | 命令                                       |
| --------------------------- | ---------------------------------------- |
| 每1分钟执行一次command             | \* \* \* \* \* command                   |
| 每小时的第3和第15分钟执行              | 3,15 \* \* \* \* command                 |
| 在上午8点到11点的第3和第15分钟执行        | 3,15 8-11 \* \* \* command               |
| 每隔两天的上午8点到11点的第3和第15分钟执行    | 3,15 8-11 \*/2 \* \* command             |
| 每个星期一的上午8点到11点的第3和第15分钟执行   | 3,15 8-11 \* \* 1 command                |
| 每晚的21:30重启smb               | 30 21 \* \* \* /etc/init.d/smb restart   |
| 每月1、10、22日的4 : 45重启smb      | 45 4 1,10,22 \* \* /etc/init.d/smb restart |
| 一月一号的4点重启smb                | 0 4 1 jan \* /etc/init.d/smb restart     |
| 每小时执行/etc/cron.hourly目录内的脚本 | 01 \* \* \* \*  root run-parts /etc/cron.hourly |

### 使用注意事项

#### 注意环境变量问题

有时我们创建了一个crontab，但是这个任务却无法自动执行，而手动执行这个任务却没有问题，这种情况一般是由于在crontab文件中没有配置环境变量引起的。

在crontab文件中定义多个调度任务时，需要特别注意的一个问题就是环境变量的设置，因为我们手动执行某个任务时，是在当前shell环境下进行的，程序当然能找到环境变量，而系统自动执行任务调度时，是不会加载任何环境变量的，因此，就需要在crontab文件中指定任务运行所需的所有环境变量，这样，系统执行任务调度时就没有问题了。

不要假定cron知道所需要的特殊环境，它其实并不知道。所以你要保证在shelll脚本中提供所有必要的路径和环境变量，除了一些自动设置的全局变量。所以注意如下3点：

- 脚本中涉及文件路径时写全局路径；

- 脚本执行要用到java或其他环境变量时，通过source命令引入环境变量，如：

  

  ```
  cat start_cbp.sh
  #!/bin/sh
  source /etc/profile
  export RUN_CONF=/home/d139/conf/platform/cbp/cbp_jboss.conf
  /usr/local/jboss-4.0.5/bin/run.sh -c mev &
  ```

- 当手动执行脚本OK，但是crontab死活不执行时。这时必须大胆怀疑是环境变量惹的祸，并可以尝试在crontab中直接引入环境变量解决问题，如：

  ```
  0 * * * * . /etc/profile;/bin/sh /var/www/java/audit_no_count/bin/restart_audit.sh
  ```

#### 注意清理系统用户的邮件日志

每条任务调度执行完毕，系统都会将任务输出信息通过电子邮件的形式发送给当前系统用户，这样日积月累，日志信息会非常大，可能会影响系统的正常运行，因此，将每条任务进行重定向处理非常重要。

例如，可以在crontab文件中设置如下形式，忽略日志输出：

```
0 */3 * * * /usr/local/apache2/apachectl restart >/dev/null 2>&1
```

“/dev/null 2>&1” 表示先将标准输出重定向到/dev/null，然后将标准错误重定向到标准输出，由于标准输出已经重定向到了/dev/null，因此标准错误也会重定向到/dev/null，这样日志输出问题就解决了。

#### 系统级任务调度与用户级任务调度

系统级任务调度主要完成系统的一些维护操作，用户级任务调度主要完成用户自定义的一些任务，可以将用户级任务调度放到系统级任务调度来完成（**不建议这么做**），但是反过来却不行。

root用户的任务调度操作可以通过

```
crontab –uroot –e
```

来设置，也可以将调度任务直接写入/etc/crontab文件，需要注意的是，如果要定义一个定时重启系统的任务，就必须将任务放到/etc/crontab文件，即使在root用户下创建一个定时重启系统的任务也是无效的。

#### 其他注意事项

新创建的cron 任务，不会马上执行，至少要过2分钟才执行。如果重启cron则马上执行。当 crontab 突然失效时，可以尝试/etc/init.d/crond restart解决问题。或者查看日志看某个job有没有执行/报错

```
tail -f /var/log/cron
```

千万别乱运行

```
crontab -r
```

它从Crontab目录（/var/spool/cron）中删除用户的Crontab文件。删除了该用户的所有crontab都没了。

在crontab中%是有特殊含义的，表示换行的意思。如果要用的话必须进行转义\%，如经常用的

```
date '+%Y%m%d'
```

在crontab里是不会执行的，应该换成

```
date '+\%Y\%m\%d'
```



Read More: 

> [每天一个linux命令（50）：crontab命令](http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)

