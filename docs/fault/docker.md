# 记录docker故障


#### 故障现象
> docker容器启动后对应的web页面无法打开，经过测试发现能ping通容器的ip地址，但80端口不通，在容器里nginx进程是开启的

```
[root@xxxx ~]# telnet  172.17.0.5 80  >>> port 不通
Trying 172.17.0.5...
telnet: connect to address 172.17.0.5: Connection refused
[root@xxxx ~]# ping 172.17.0.5   >>>> ping 正常
PING 172.17.0.5 (172.17.0.5) 56(84) bytes of data.
64 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from 172.17.0.5: icmp_seq=2 ttl=64 time=0.297 ms
```
#### 排障过程
**虚拟机网络信息**
```
[root@xxxxx ~]# ip a     >> ip地址
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether fa:16:3e:38:55:a2 brd ff:ff:ff:ff:ff:ff
    inet 10.x.x.x/22 brd x.x.x.x scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe38:55a2/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:3e:55:d1:25 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:3eff:fe55:d125/64 scope link 
       valid_lft forever preferred_lft forever
71: veth62bb9d8@if70: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP 
    link/ether 86:6b:30:b2:23:49 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::846b:30ff:feb2:2349/64 scope link 
       valid_lft forever preferred_lft forever
[root@xxxx ~]# ip route   >> 查看路由器
default via 10.x.x.1 dev eth0 proto static metric 100 
x.x.x.x/22 dev eth0 proto kernel scope link src x.x.x.x metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
[root@xxx ~]# arp -a   >> **查看arp**
? (172.17.0.5) at 02:42:ac:11:00:05 [ether] on docker0
? (x.x.x.x) at fa:16:3e:07:fe:fd [ether] on eth0
? (172.17.0.3) at 02:42:ac:11:00:03 [ether] on docker0
gateway (x.x.x.x) at 00:00:11:11:22:22 [ether] on eth0
```
**容器网络信息**
```
root@6d75e1a50c66:/opt/django/ctools# ip a  >> 查看ip地址
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
82: eth0@if83: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:05 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.5/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
root@6d75e1a50c66:/opt/django/ctools# ip route   >> 路由
default via 172.17.0.1 dev eth0 
172.17.0.0/16 dev eth0  proto kernel  scope link  src 172.17.0.5

root@7ce4350d321f:/opt/django/ctools# ps -ef | grep -i nginx   >>>>> nginx进程是启动的
root        14     0  0 May26 ?        00:00:00 nginx: master process nginx
www-data    15    14  0 May26 ?        00:00:50 nginx: worker process
www-data    16    14  0 May26 ?        00:00:52 nginx: worker process
www-data    17    14  0 May26 ?        00:00:55 nginx: worker process
www-data    18    14  0 May26 ?        00:00:51 nginx: worker process
root@7ce4350d321f:apt-get install net-tools 
root@7ce4350d321f:/opt/django/ctools# netstat -tlnp    >>>>> 80端口不存在
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:8001          0.0.0.0:*               LISTEN      36/uwsgi        
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      27/redis-server 
tcp        0      0 0.0.0.0:5555            0.0.0.0:*               LISTEN      21/python       
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      20/sshd         
tcp6       0      0 :::5555                 :::*                    LISTEN      21/python       
tcp6       0      0 :::22                   :::*                    LISTEN      20/sshd  
```
**可能原因**：从现象来看，`nginx`进程是启动的，但是80端口没有起来,查看`nginx error`日志无报错信息，再溯源查看nginx文件是否正常，发现nginx文件没有挂载

**根因：** 在容器启动的时候需要挂载nginx配置文件，但是该目录下的配置文件突然不存在了

参考资料：[同一宿主机内的docker通信](https://keyu.site/2019/08/16/dockerNet/)











