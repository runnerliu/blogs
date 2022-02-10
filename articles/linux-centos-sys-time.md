## CentOS 系统时间校对

### CentOS系统时间、时区查看

查看系统时间：date

查看硬件时间：hwclock

查看系统详细时间：timedatectl

查看系统所有时区：timedatectl list-timezones

将硬件时钟调整为与本地时钟一致：timedatectl set-local-rtc 1（其中 0 表示UTC时间）

设置系统时区为上海：timedatectl set-timezone Asia/Shanghai

### 校正系统时间

确认是否安装utpdate工具：`yum list installed|grep ntpdate`

如未安装，需执行 `yum -y install ntpdate` 

执行 `ntpdate -u  pool.ntp.org` 进行时间校正

