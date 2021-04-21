#### 收发包整体流程

**内核空间提供给用户空间的接口socket抽象接口**，在 Linux 内核中，系统使用`sk_buff`数据结构对数据包进行存储和管理。在数据包接收过程中，该数据结构从网卡驱动收包开始，一直贯穿到内核网络协议栈的顶层，直到用户态程序从内核获取数据。
![](https://files.mdnice.com/user/4251/4b9b983b-c5d4-4a93-97ad-94fc8009f9a0.png)

- 用户态(User Space)程序Client向另外一台主机上的Server发送数据，**需要通过调用内核态(Kernel Space)提供给用户态的Socket抽象层接口**发送数据；

- Socket抽象层接口收到用户态数据后，向下交给传输层接口(TCP或UDP)；
- 传输层负责创建sk_buff,并将用户数据(应用层数据)填充到缓冲区，做合法性检查后，添加传输层头部，并通过网络层注册的接口将数据包交给网络层处理；
- 网络层收到传输层数据包后，会查询路由表，决定数据包去向，如果是需要发出的数据包，会填充网络层头部，并交给内核虚拟网络接口设备的发送队列中；
- 虚拟网络接口从发送队列获取数据，调用对应网卡驱动发送数据；

Server端接收到数据后，会按照相反的过程从网卡驱动中将数据包一层层上交，直到通过socket抽象层接口将用户数据上交到用户态server进程处理。

#### 网络层(IPV4)收发包流程
linux内核网络层收发包大概调用流程(**这是站在函数的角度**)
![](https://files.mdnice.com/user/4251/737fd18e-9696-41c2-b144-35cae6e05975.png)

- 从图中可以看到，**ip_rcv**函数为**网络层向下层开放的入口**，数据包通过该函数进入网络层进行处理，该函数主要对上传到网络层的数据包进行前期合法性检查，通过后交给**netfilter**的钩子节点。
- 绿色方框内的**IP_PRE_ROUTING**为netfilter框架的Hook点，该节点会根据预设的规则对数据包进行判决并根据判决结果做相关的处理，比如执行**NAT转换**
- **IP_PRE_ROUTING**节点处理完成后，数据包将交由**ip_rec_finish**处理，该函数根据路由判决结果，决定数据包是交由本机上层应用处理，还是需要进行转发；如果是交由本机处理，则会交由**ip_local_deliver**走本地上交流程；如果需要转发，则交由**ip_forward**函数走转发流程；
- 在数据包上交本地的流程中，**IP_LOCAL_INPUT**节点用于监控和检查上交到本地上层应用的数据包，该节点是linux防火墙重要生效节点之一；
- 在数据包转发流程中，netfilter框架的**IP_FORWARD**节点会对转发数据包进行检查过滤；
- 而对于本机上层发出的数据包，网络层通过注册到上层的**ip_local_out**函数接收数据处理，处理OK进一步交由**IP_LOCAL_OUT**节点检测；
- 对于即将发往下层的数据包，需要经过IP_POST_ROUTING节点处理；网络层处理结束，通过*dev_queue_xmit*函数将数据包交由 Linux 内核中虚拟网络设备做进一步处理，从这里数据包即离开网络层进入到下一层；


#### netfilter
> 在linux系统中，如果我们需要对网络数据流程做一些特殊处理，那就是netfilter模块，该模块负责涉及到所有的网络流程相关的操作。

在设计好linux网络处理层后，自然我们需要提供一些接口给用户态的用户去编辑规则，实现用户的自定义。而这个功能交给了`netfilter`框架实现(如数据包过滤，网络地址跟踪)，主要就是在处理数据包的关键流程中定义**一系列钩子函数(Hook点)**

![](https://files.mdnice.com/user/4251/0aff664f-664e-4842-8aaa-4f903fd3775d.png)


##### netfilter hook点
我们不考虑实际的处理函数，仅仅看**netfilter**的钩子节点在网络数据流程中的位置见下图：

![](https://files.mdnice.com/user/4251/9769c3ce-bc2f-43bc-bebd-afcb10e0c5cf.png)

路由决定方向，不同的方向涉及到的钩子节点就完全不一样了：

- 发往本地：NF_INET_PRE_ROUTING-->NF_INET_LOCAL_IN
- 转发：NF_INET_PRE_ROUTING-->NF_INET_FORWARD-->NF_INET_POST_ROUTING
- 本地发出： NF_INET_LOCAL_OUT-->NF_INET_POST_ROUTING

##### iptable工具

iptables在用户态提供了表格和链的概念，四表五链


![](https://files.mdnice.com/user/4251/136fc436-47ce-4d9e-ab50-3961d3097c59.png)

iptalbes中每个表格的作用不同，以我们比较常用的filter表为例，其主要起到数据包过滤和拦截作用，包含INPUT，FORWARD 和 OUTPUT 三个链,INPUT 链是在NF_INET_LOCAL_IN节点中生效，FORWARD 链是在NF_INET_FORWARD节点，OUTPUT 链则是在NF_INET_LOCAL_OUT节点生效。

以`iptables`指令为例:
```
iptables -t filter -A INPUT -s 172.16.0.0/16 -p udp --dport 53 -j DROP
```
从定义我们看出，filter表input链执行符合条件的丢弃动作，而在linux内核中，这一个指令会在netfilter网络层`NF_INET_LOCAL_IN`节点生成处理操作，凡是经过这个钩子节点的数据包,**在前面规则都通过的情况下，都必须经过这一规则的检查**

iptables是netfilter提供api接口，用户通过操作iptable的数据最后作用于数据包处理流程中，INPUT钩子定义在数据包处理流中位置，而表项定义了匹配规则的内容即执行动作。

#### 感受

当我们定义好一套合理的数据处理流程后，如果无法对其进行控制的话，那生命力会大大降低，所以我们不得不提供具体api接口给用户自定义数据表项。往往实现上可能采用钩子函数(**像AOP**)，在关键节点处挂入钩子，对报文进行处理。




