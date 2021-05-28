#### 服务网格是什么
服务网格是用于处理服务间通信的专用**基础设施层**，主要负责服务之间的**网络调用、限流、熔断等**，与spring cloud实现的功能是一样的，也就是如果通过服务网格来实现这些基础设施功能，程序就不用在依赖`Spring Cloud`等框架，只需要交给服务网格就行。

#### 服务网格发展历史
- 从最原始的主机之间直接使用网线相连
- 网络层的出现
- 集成到应用程序内部的控制流
- 分解到应用程序外部的控制流
- 应用程序的中集成服务发现和断路器
- 出现了专门用于服务发现和断路器的软件包/库，如 Twitter 的`Finagle`和`Facebook`的`Proxygen`，这时候还是集成在应用程序内部
- 出现了专门用于服务发现和断路器的开源软件，如 `Netflix OSS`、`Airbnb` 的 sy`napse` 和 nerve
- 最后作为微服务的中间层服务网格出现

同类工具：服务网格、spring cloud 、oss

#### 为什么服务网格会出现？

工具包，已经能实现**限流熔断**等应用需求，但这些实现都有一定的局限性。

- 只适用于java语言
- 需要在代码中绑定，有侵入性
- 没有统一的管理平台

**功能的实现很重要，但统一的管理更重要。**

而服务网格针对云平台的解决方案，就解决了以上问题。

Service Mesh：Service Mesh是一种**非入侵、透明化的微服务治理框架**。作为服务与服务直接通信的透明化管理框架，**Service Mesh不限制服务开发语言**、使用轻量级的通信协议（HTTP、gRPC等），并插件式的提供各类功能，如服务发现、负载均衡、智能路由、流量管控、性能分析等等，换而言之，用户通过Service Mesh得以用简单的方式获取高级的功能。

Service Mesh由于可以用更高效和统一的方式在新的微服务和容器环境中管理所有服务，可以说，它是是企业扩展、保护和监控应用程序的最佳方式。


#### 服务网格的架构图

![](https://files.mdnice.com/user/4251/595c0c4e-87ca-43e2-90d4-b110b45454eb.png)
![](https://istio.io/latest/zh/docs/ops/deployment/architecture/arch.svg)

从架构中，我们可以看出`node`与`api server`之间出现了一层服务网格层，任何流量都需要经过服务网格转发。

为了实现统一管理，必然会增加控制层，同时提供全局可视化工具，如`kiali`。



#### 可视化工具 kiali


参考资料

- [istio-handbook](https://jimmysong.io/istio-handbook/concepts/what-is-service-mesh.html)  写的很好
- [腾讯云容器团队内部Istio专题分享](https://www.servicemesher.com/blog/istio-the-king-of-service-mesh/)
- [服务网格发展历史](https://jimmysong.io/istio-handbook/concepts/what-is-service-mesh.html)
