#### 链接
- [Pod生命周期](https://www.icode9.com/content-4-714365.html)
- [pod生命周期2](https://jimmysong.io/kubernetes-handbook/concepts/pod-lifecycle.html)
![image](https://user-images.githubusercontent.com/30467613/115977253-c994e800-a5a8-11eb-83c1-ce7771e71067.png)

#### 启动顺序
- 会先启动pause根容器
- 然后顺序执行一系列的init c(初始化容器)【如果配置了的话】
- 然后start主容器Main C
- readiness检测到Main C启动完成后会把pod的状态改为running(如果配置了的话)
- 这之后生存检测liveiness会持续监测容器的状态(主容器启动后的监控应用自身的状态) 也需要配置

`init`(生命周期只在于初始化阶段)  `readiness`(生命周期只在main c容器启动阶段)  `liveiness`(主容器启动后的健康状态)

- `livenessProbe`：指示容器是否正在运行。如果存活探测失败，则 kubelet 会杀死容器，并且容器将受到其 重启策略 的影响。如果容器不提供存活探针，则默认状态为 Success。
- `readinessProbe`：指示容器是否准备好服务请求。如果就绪探测失败，端点控制器将从与 Pod 匹配的所有 Service 的端点中删除该 Pod 的 IP 地址。初始延迟之前的就绪状态默认为 Failure。如果容器不提供就绪探针，则默认状态为 Success

他们应用在什么场景？

`livenessProbe探针`主要是监控应用程序内部接口运行是否正常，很多时候应用提供的服务是异常的，但进程却是好的(并没有崩溃)，这时候就需要探针强制重启pod。

`readinessProbe探针` 目的是确保服务真正起来了，可以提供服务的，而不是pod是run，但服务未好，在这个时候流量通过service导入过来就会访问异常。


