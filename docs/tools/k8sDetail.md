#### 概述
> k8s已成为pass平台的事实标准，越来越多的企业将应用发布在k8s上面，通过k8s管理企业中的大量应用，k8s本身是为应用服务的，所以很多功能的设计都要站在对开发的友好角度来思考。

#### 功能
应用的水平扩缩容，应用的滚动升级，基于自定义指标实现扩容缩容等。

#### 概念
k8s里面涉及到的概念很多，但真正的实体(应用承载体)只有pod，而其他概念的出现都是为如何访问到pod做设计

应用实体pod

- **pod** k8s里面最小的单元，而一个容器或者多个相关容器的集合，对标现在的微服务，也就是说pod是应用的承载体，应用都部署在pod里面。而我们的应用有很多种类型，如job，stateful，stateless，deamon等，

其他

- **service** 为了实现高可用高负载，同类型pod往往会起很多个，这就需要一个汇总模块，将这些pod管理起来，这就是service要做的事情

- **网络插件** pod之间如何通信？k8s只定义网络规范，具体的实现交给对应的插件

- **内部dns** 
- 
- **ingress** 将k8s集群对外发布，是外部访问k8s集群的入口
- 
- **volume** 容器是无状态的，通过volume的方式实现持久化

#### 网络
pod---cni(相当pod的网关)---ipvs---eth0，常用插件有Flannel，Calico，Weave，不同插件在具体实现上有一些差异，比如Flannel通过overlay的方式实现，而Calico通过bgp的方式实现

[超强指南！Kubernetes跨节点网络通信之Flannel](https://mp.weixin.qq.com/s/uJR4YmUuSCjgEi-VNkTLnA)

[Kubernetes网络全解：机制、方法、实操的超强指南](https://mp.weixin.qq.com/s/YO-MPJFei6flcvQdktQRww)

#### service暴露给外部集群



