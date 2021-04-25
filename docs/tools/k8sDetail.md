#### 概述
> k8s已成为pass平台的事实标准，越来越多的企业将应用发布在k8s上面，通过k8s管理企业中的大量应用，k8s本身是为应用服务的，所以很多功能的设计都要站在对开发的友好角度来思考。

#### 功能
应用的水平扩缩容，应用的滚动升级，基于自定义指标实现扩容缩容,资源限额。

#### 概念
k8s里面涉及到的概念很多，但真正的实体(应用承载体)只有pod，而其他概念的出现都是为如何访问到pod做设计


应用实体pod

- **pod** k8s里面最小的单元，而一个容器或者多个相关容器的集合，对标现在的微服务，也就是说pod是应用的承载体，应用都部署在pod里面。而我们的应用有很多种类型，如job，stateful，stateless，deamon等，

其他

- **service** 为了实现高可用高负载，同类型pod往往会起很多个，这就需要一个汇总模块，将这些pod管理起来，这就是service要做的事情

- **网络插件** pod之间如何通信？k8s只定义网络规范，具体的实现交给对应的插件

- **内部dns** 
- **ingress** 将k8s集群对外发布，是外部访问k8s集群的入口
- **volume** 容器是无状态的，通过volume的方式实现持久化

层次关系： cluster----project----namespace---pod

#### 架构
* [k8s架构](k8s.md)
* [k8s外部访问数据流](dataflow.md)

#### pod生命周期
* [pod生命周期](pod_life.md)
#### 网络
pod---cni(相当pod的网关)---ipvs---eth0，常用插件有Flannel，Calico，Weave，不同插件在具体实现上有一些差异，比如Flannel通过overlay的方式实现，而Calico通过bgp的方式实现

[超强指南！Kubernetes跨节点网络通信之Flannel](https://mp.weixin.qq.com/s/uJR4YmUuSCjgEi-VNkTLnA)

[Kubernetes网络全解：机制、方法、实操的超强指南](https://mp.weixin.qq.com/s/YO-MPJFei6flcvQdktQRww)

#### service
[12张图，带你轻松理解Kubernetes Service](https://mp.weixin.qq.com/s/dAujOyQLJOuzSu-tamH7zA)

`hostNetwork hostport` 是直接将`pod`暴露给外部环境，`NodePort，LoadBalancer，Ingress`将`service`暴露到外部环境，实际使用中主要是通过ingress对外暴露(pod-cluster-ingress)

[Kubernetes的三种外部访问方式](http://dockone.io/article/4884)

[kubenretes中的Pod和Serivce的几种方式](https://jimmysong.io/blog/accessing-kubernetes-pods-from-outside-of-the-cluster/)

#### 存储
* [存储类型](k8s_data.md)

#### 服务网格istio
> 服务网格简单的理解就是应用程序中非功能模块，但又很重要，如流量管理，服务降级，故障注入等一切额外的功能，如果我们自己在写代码要实现这些功能一般会采用aop思想，但如果每个工程师
在项目中都实现一遍的话，那对于企业来说成本非常高，那就抽象出来，让这些非功能模块的事情都交给服务网格来做(istio)

* [服务网格istio](https://jimmysong.io/kubernetes-handbook/usecases/istio.html)


#### 常用命令

获取基础：kubectl get [pods|services|ingress] -A  -o wide

获取所有运行的容器：kubectl get pods --all-namespaces -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' | sort

kubectl logs grafana-7b7bdb59b7-7hhh6 -n grafana

查看Ingress Controller日志
kubectl -n ingress-nginx logs -l app=ingress-nginx

[常用命令](https://www.huaweicloud.com/articles/7d96cb778b661628f5c522e0fd1dda8d.html)

#### 排障思路

* [k8s排障思路](k8s_error_guide.md)


#### 参考连接





