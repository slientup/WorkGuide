


#### 入门资料
[Kubernetes 中文指南/云原生应用架构实践手册](https://jimmysong.io/kubernetes-handbook/)

#### 各个组件的产生背景
参考
[写给大家看的“不负责任” K8s 入门文档](https://mp.weixin.qq.com/s/9NAUvPbxqR9qlL9_Gd6YIA)

容器编排系统的实现:我们的服务器就应该有一部分实现编排系统，有一部分实现业务容器，我们把运行编排系统的容器称作为master节点，运行业务的称作为worker节点

既然master节点负责管理服务器集群，那么它一定会提供相关的管理接口，一个是方便运维管理人员对集群进行相关的操作，另一个是给负责跟worker节点进行相互通信，
如进行资源的分配、网络的管理等。

我们把master节点提供管理接口组件称作为`kube apiserver`，对应的还有两个与apiserver通信的客户端
- 一个是提供给k8s集群运维的管理人  `kubectl`
- 一个提供给worker使用的节点  `kubelet`

现在集群的管理人员、master、worker之间可以通信了，比如说运维管理人员通过kubectl向master节点下发一条命令,"用淘车网用户中心2.0的镜像创建1000个容器"，
master收到这条指令后，会根据集群里面worker资源信息进行一个资源计算，算出这1000个容器应该在哪些worker中创建，然后把创建指令下发到对应的worker，我们
把这个负责调度的组件称作为`kube scheduler`。

我们的master是如何知道各个worker的资源消耗和容器运行情况的？

我们可以通过worker上面的`kubelet周期性的主动上报节点`的资源和运行情况，然后master把这个数据存储起来，后面我们就可以做调度和容器的管理使用了，至于数据怎么储存，我们可以选择写文件、写db等等。不过有个开源的存储系统`etcd`，满足我们对数据一致性和高可用的要求，同时安装简单，性能又好，我们就选它吧

比如前面所说的，我们使用淘车网用户中心 2.0 版本的镜像创建了 1000 个容器，其中有 5 个容器都是运行在 A 这个 worker 节点上，那如果 A 这个节点突然出现
了硬件故障，导致节点不可用了，这个时候 master 就要把 A 从可用 worker 节点中摘除掉，并且还需要把原先运行在这个节点上的 5 个用户中心 2.0 的容器重新
调度到其他可用的`worker`节点上，使得我们用户中心 2.0 的容器数量能够重新恢复到1000个，并且还需要对相关的容器进行网络通信配置的调整，使得容器间的
通信还是正常的.

我们把这一系列的组件称为控制器，比如节点控制器、副本控制器、端点控制器等等，并且为这些控制器提供一个统一的运行组件，
称为控制器管理器（`kube-controller-manager`）.



那 master 又该如何实现和管理容器间的网络通信呢？

- 首先每个容器肯定需要有一个唯一的 ip 地址，通过这个 ip 地址就可以互相通信了
- 但是彼此通信的容器有可能运行在不同的 worker 节点上，这就涉及到 worker 节点间的网络通信，因此每个 worker 节点还需要有一个唯一的 ip 地址；
- 但是容器间通信都是通过容器 ip 进行的，容器并不感知 worker 节点的 ip 地址，因此在 worker 节点上需要有容器 ip 的路由转发信息，
  我们可以通过 iptables、ipvs 等技术来实现；
- 那如果容器 ip 变化了，或者容器数量变化了，这个时候相关的 iptables、ipvs 的配置就需要跟着进行调整，所以在 worker 节点上我们需要
一个专门负责监听并调整路由转发配置的组件，我们把这个组件称为 kube proxy。

我们已经解决了容器间的网络通信，但是在我们编码的时候，我们希望的是通过域名或者 vip 等方式来调用一个服务，而不是通过一个可能随时会变化的容器 ip。
因此我们需要在容器 ip 之上再封装出一个`service `的概念，这个 service 可以是一个集群的 `vip，也可以是一个集群的域名`，为此我们还需要一个集群内部的 
DNS 域名解析服务.对外提供统一的服务

另外虽然我们已经有了 kubectl，可以很愉快的和 master 进行交互了，但是如果有一个 web 的管理界面，这肯定是一个更好的事情。此处之外，我们可能还希望看到
容器的资源信息、整个集群相关组件的运行日志等等

像 `DNS、web 管理界面、容器资源信息、集群日志`，这些可以`改善我们使用体验的组件`，我们统称为`插件`.

至此，我们已经成功构建了一个容器编排系统，下面我们来简单总结一下上文提到的各个组成部分：

`Master 组件`：kube-apiserver、kube-scheduler、etcd、kube-controller-manager；  
`Node 组件 ：kubelet、kube-proxy；   
`插件 ：DNS、用户界面 Web UI、容器资源监控、集群日志。   
- ![k8s组件](https://github.com/slientup/WorkGuide/blob/master/k8s.png)



































