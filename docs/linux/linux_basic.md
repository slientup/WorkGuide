### 权限命令

#### sudo
配置文件路径：`/etc/sudoers`
```
## Allow root to run any commands anywhere
root ALL=(ALL)ALL
# admin ALL=(ALL) NOPASSWD:ALL  举例
用户名登陆者的来源主机名=授权角色（可切换的身份）是否需要密码可执行命令... （不可执行用!）
```

#### su
> 切换到其他用户

在某个用户下执行某条命令 `su zxzeng -c "command"`

### 系统信息

#### uname
```
-a, --all                  按顺序打印全部信息，如果 -p 和 -i 的信息是未知，那么省略。
-s, --kernel-name          打印内核名称。
-n, --nodename             打印网络节点主机名称。
-r, --kernel-release       打印内核release。
-v, --kernel-version       打印内核版本。
-m, --machine              打印机器名称。
-p, --processor            打印处理器名称。
-i, --hardware-platform    打印硬件平台名称。
-o, --operating-system     打印操作系统名称
```

#### dmidecode

-q 显示硬件系统部件 - (SMBIOS / DMI) 

cat /proc/cpuinfo 显示CPU info的信息

cat /proc/meminfo 校验内存使用

cat /proc/swaps 显示哪些swap被使用

cat /proc/net/dev 显示网络适配器及统计

cat /proc/mounts 显示已加载的文件系统


#### 时钟
linux 存在两个时钟，分别是硬件时钟和系统时钟，大多数操作系统的时钟管理方式如下：
- 启动时根据硬件时钟设置系统时钟
- 维护准确的系统时钟
- 关机时根据系统时钟设置硬件时钟

`date`  查看当前时间

`date '+%s'` 当前时间转时间戳

`timedatectl set‐time '2022‐10‐10 11:11:11'`   修改本地时钟

`timedatectl set‐timezone Asia/Shanghai`  设置时区

`hwclock ‐‐set ‐‐date "21 Oct 2020 21:17"`  设置硬件时钟

`hwclock -w`  将时间修改保存到BIOS

### 系统

#### top
```
PID 进程 ID

USER 进程所有者的用户名

PR 任务优先级

NI nice值。数值越小表示优先级越高，数值越大表示优先级越低

VIRT 进程使用的虚拟内存总量，单位:kb  VIRT=SWAP+RES

RES 进程使用的、未被换出的物理内存大小，单位：kb  RES=CODE+DATA

SHR 共享内存大小， 单位：kb

S 进程状态

D= 不可中断的睡眠状态

S= 睡眠

T= 跟踪 / 停止

Z= 僵尸进程

%CPU 上次更新到现在的CPU时间占用百分比

TIME+ 进程使用的CPU时间总计，精确到1/100秒

COMMAND 命令名 / 命令行 

```

#### ps aux 
```
USER: 该进程属于哪个使用者的账号？
PID: 该进程的进程ID号
%CPU：该进程使用掉的CPU资源百分比
%MEM：该进程所占用的物理内存百分比
VSZ：该进程使用掉的虚拟内存量(kb)
RSS: 该进程占用的固定内存量(kb)
TTY: 该进程是在那个终端机上面运作。
STAT: 该程序目前的状态，主要状态有：
R：该程序目前正在运行，或者是可被运行的
S：该程序目前正在睡眠中，但可被某些信号(signal)唤醒
T：该程序目前正在侦测或者停止了
Z：该程序应该已经终止，但是其父程序却无法正常终止他，造成僵尸程序的状态。
START：该进程被触发启动的时间
TIME：该进程实际使用CPU运作的时间
COMMAND：该程序的实际执行指令
```

#### free
```
powerop@VMS02911:~$ free
             total       used       free     shared    buffers     cached
Mem:      24692640    7168752   17523888          0    1229844    2619808
-/+ buffers/cache:    3319100   21373540
Swap:      3997692          0    3997692
```
buffer与cached的区别：
- A buffer is something that has yet to be “written” to disk. 针对写场景，当向disk写入数据的时候，先写入到buffer
- A cache is something that has been “read” from the disk and stored for later use. 针对读场景，在disk中读取数据后并放入cache中以便下次使用

#### iostat -k 2  
> 磁盘io的统计信息，间隔2s刷新一次
```
powerop@VMS02911:~$ iostat -k 2
Linux 3.8.0-44-generic (VMS02911)       03/03/2021      _x86_64_        (4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.16    0.24    0.28    0.00    0.00   99.33

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               2.33         0.10        26.33    1970644  543577779
dm-0              5.19         0.10        26.33    1964805  543543548
dm-1              0.00         0.00         0.00       1208          0
```

#### vmstat
> vmstat可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况
```
powerop@VMS02911:~$ vmstat 2 3  每2秒采集一次主机状态，共采集3次
procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 2  0      0 17520408 1229852 2621868    0    0     0     7    0    0  0  0 99  0
 0  0      0 17520496 1229852 2621872    0    0     0    40  430 1055  1  0 99  0
 0  0      0 17520276 1229852 2621908    0    0     0    50  455 1082  0  0 99  0
```

#### shutdown
关机  `shutdown -h now`

重启  `shutdown -r now`

#### systemctl

启动服务  `systemctl start crond`

重启服务  `systemctl restart crond`

重载服务 `systemctl reload crond`

查看服务运行状态  `systemctl status crond`

停止服务 `systemctl stop crond`

设置开机自启动 `systemctl enable mysql.service`

关闭开机启动 `systemctl disable mysql.service`

列出所有可用单元 `systemctl list-unit-files`

列出所有运行中的单元  `systemctl list-units`

列出所有失败的单元  `systemctl -failed`

#### journalctl
> Systemd 统一管理所有 Unit 的启动日志。带来的好处就是 ，可以只用journalctl一个命令，查看所有日志（内核日志和 应用日志）

返回本次启动的日志  `journalctl`

只看内核日志   `journalctl -k`

本次启动的日志  `journalctl -b`


#### df 
> 查看磁盘空间
查看磁盘使用情况  `df -h`    `df -ThP`

查询inode  `df -ihP`


#### du
> 查看目录下文件大小

查看root目录文件大小  `du -h /root`

查看inode占用  `du --inode /root/`

目录深度 `du ‐‐max‐depth=1 ‐h /root`


#### yum

查找可安装包 `yum list | grep `nginx` yum search nginx`

查看版本  `yum list docker-ce --showduplicates`

安装  `yum install -y nginx`

卸载  `yum remove -y nginx`

只下载不安装，指定下载目录  `yum install ‐‐downloaddir=/home/nginx_install_package ‐‐downloadonly nginx`

搜索历史操作  `yum history list nginx`

重做  `yum history undo N`

回滚  `yum history redo N`

配置yum源  `/etc/yum.repos.d/`

#### npm 

查看已安装的npm包  `rpm -qa`

浏览npm包的详细 `rpm -ql nginx`

卸载npm包  `rpm -e nginx`

#### tar

打包压缩  `tar -czvf rep1.tar.gz repo_bak/`

解压  `tar -zxvf rep.tar.gz`


#### 主机各类状态查询 /proc
```linux
/proc/cpuinfo cpu的信息
/proc/stat 所有的CPU活动信息
/proc/meminfo RAM使用的相关信息
/proc/vmstat 虚拟内存统计信息
/proc/swaps 交换空间的使用情况
/proc/devices 已经加载的设备并分类
/proc/filesystems 内核当前支持的文件系统类型
/proc/mdstat 多硬盘，RAID配置信息(md=multiple disks)
/proc/pci 系统中的PCI设备列表
/proc/diskstats 取得磁盘信息
/proc/mounts 系统中使用的所有挂载
/proc/net 网卡设备信息
/proc/tty tty设备信息
/proc/fs 文件系统信息
/proc/tty tty设备信息
```


![](https://imgkr2.cn-bj.ufileos.com/8ef3334e-ead5-45e8-a821-7e90960ccca6.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=NWMtf8Z56Bi%252Bm91xBKTm1Jr8OkQ%253D&Expires=1614916804)



![](https://static01.imgkr.com/temp/90e9fbeba831439aa02ef5048ee0d481.png)

### 文件和目录
#### vim

#### cd

进入home目录 `cd ~`

进入上一级  `cd ..`

进入上一级的上一级 `cd ../..`

返回上一次目录  `cd -`

#### mkdir

创建目录  `mkdir -p /zxzeng/test1`

#### cp
cp fileA fileB 
```
-r: 若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件

-a: 此选项通常在复制目录时使用，它保留链接，文件属性，并复制目录下的所有内容。其作用等于dpR参数组合

-f: 覆盖已经存在的目标文件而不给出提示

-i: 与-f选项相反，在覆盖目标文件之前给出提示，要求用户确认是否覆盖

-p: 除复制文件的内容外，还把修改时间和访问权限也复制到新文件中

```

#### mv
剪切或重命名

源文件比目标文件新时才执行更新   `mv ‐uv *.txt /home/office`

不要覆盖任何已存在的文件 `mv ‐vn *.txt /home/office`

无条件覆盖已经存在的文件 `mv ‐f *.txt /home/office`


### 文本处理
#### sort
以数字排序  `sort -n`
去重排序  `sort -u`
指定列排序  `cat 1.txt | sort ‐k1.4`

#### uniq

在文件中找出重复的行  `sort file.txt | uniq -d`

统计各行在文件中出现的次数 `sort file.txt | uniq ‐c`

#### cut 

截取第一和第四个字符  `cat 1.txt | cut -c 1,4`

指定分隔符 `cut -d '分隔符'`

截取字段 `cut -f 1`

截取字节 `cut -b 1`

#### grep

#### sed



#### awk










### 查看文件内容
`cat` 从第一行开始显示
`tac` 从最后一行开始显示
`more` 分页，只能往后
`less` 分页，前后都可以
`head` 显示头几行
`tail` 显示最后几行



























 




















### 网络
#### ifconfig 
> 看更多的物理网卡信息 

第一行：连接类型：Ethernet（以太网）HWaddr（硬件mac地址）。

第二行：网卡的IP地址、子网、掩码。

第三行：UP（代表网卡开启状态）RUNNING（代表网卡的网线被接上）MULTICAST（支持组播）MTU:1500（最大传输单元）：1500字节。

第四、五行：接收、发送数据包情况统计。

第七行：接收、发送数据字节数统计信息

`开启eth0` ifconfig eth0 up

`关闭eth0` ifconfig eth0 down

`配置eth0 重启失效`  ifconfig eth0 192.168.10.10 netmask 255.255.255.0

`修改mac地址` ifconfig eth0 hw ether 00:AA:BB:CC:dd:EE

`开启eth0网卡的arp协议`  ifconfig eth0 arp 

`关闭eth0网卡的arp协议`  ifconfig eth0 -arp 

`静态文件配置ip`  vim /etc/sysconfig/network‐scripts/ifcfg‐eth0
```
 TYPE=Ethernet #网卡类型
 DEVICE=eth0 #网卡接口名称
 ONBOOT=yes #系统启动时是否自动加载
 BOOTPROTO=static #启用地址协议 ‐‐static:静态协议 ‐‐bootp协议 ‐‐dhcp协议
 IPADDR=192.168.1.11 #网卡IP地址
 NETMASK=255.255.255.0 #网卡网络地址
 GATEWAY=192.168.1.1 #网卡网关地址
 DNS1=10.203.104.41 #网卡DNS地址
 HWADDR=00:0C:29:13:5D:74 #网卡设备MAC地址
 BROADCAST=192.168.1.255 #网卡广播地址

```

`重启网络`  systemctl restart network


#### route 

`显示路由表`  route -n
```
root@VMS02911:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.18.19   0.0.0.0         UG    100    0        0 eth0
192.168.18.0    0.0.0.0         255.255.255.0   U     0      0        0 eth0

# flags 字段解释：
U Up表示此路由当前为启动状态。
H Host，表示此网关为一主机。
G Gateway，表示此网关为一路由器。
R Reinstate Route，使用动态路由重新初始化的路由。
 D Dynamically,此路由是动态性地写入。
 M Modified，此路由是由路由守护程序或导向器动态修改。
 ! 表示此路由当前为关闭状态。
```

`设置默认路由(重启失效)`   route add default gw {IP‐ADDRESS} {INTERFACE‐NAME}

`添加到指定网络的路由规则`  route add ‐net {NETWORK‐ADDRESS} netmask {NETMASK} dev {INTERFACE‐NAME}

`添加一条路由，去往192.168.62网段的，都从192.168.1.1 这个网关走` route add ‐net 192.168.62.0 netmask 255.255.255.0 gw 192.168.1.1

`设置黑洞路由` route add ‐net {NETWORK‐ADDRESS} netmask {NETMASK} reject

`添加到主机的路由` route add –host 192.168.168.110 dev eth0

`删除路由` route del –host 192.168.168.110 dev eth0

`设置永久路由`
```
[root@linux-node1 network-scripts]# cat /etc/sysconfig/network-scripts/route-eth0
10.18.196.0/255.255.254.0 via 192.168.56.11 dev eth0
[root@linux-node1 network-scripts]# nmcli dev connect eth0 # 重启计算机，或者重新启用设备 eth0 才能生效。
```

#### ip 

`开启eth0` ip link set eth0 up

`关闭eth0` ip link set eth0 down

`设置eth0网卡IP地址` ip addr add 192.168.0.1/24 dev eth0

`删除eth0网卡IP地址` ip addr del 192.168.0.1/24 dev eth0

`查看路由` ip route 

`设置默认路由` ip route add default via 192.168.1.254

#### netstat

`查看网络协议丢包情况`  netstat -su
`查看路由` netstat -r

`常用` netstat -tlnp

#### dig

`dig http://www.baidu.com`


#### 查看网络流量

`iftop` 查看每个流量的实时连接

![](https://static01.imgkr.com/temp/b0b0585cec6b43abac2754542b525866.png)

`ss -nt` 查看当前通信对的流量
```
root@VMS02911:~# ss -nt
State      Recv-Q Send-Q                                                             Local Address:Port                                                               Peer Address:Port 
CLOSE-WAIT 1      0                                                                 192.168.18.166:47631                                                               10.25.60.77:9092  
ESTAB      0      0                                                                 192.168.18.166:40071                                                            192.168.93.214:4505  
ESTAB      0      0                                                                 192.168.18.166:57077                                                               10.25.60.75:9092  
CLOSE-WAIT 1      0                                                                 192.168.18.166:59020                                                              10.15.118.86:9092
```

更多的命令参考:[Linux查看网络流量](https://my.oschina.net/u/1030865/blog/3221532)







`


































