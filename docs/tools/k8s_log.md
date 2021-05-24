**k8s日志采集方案**
- Node上部署一个日志收集的agent
- 以sidebar的方式在需要收集的pod起agent
- 程序中自己写入到日志系统

**方案一的原理：**

容器化应用写入 `stdout`和`stderr`的任何数据，都会被容器引擎捕获并被重定向到某个位置。例如，Docker 容器引擎将这两个输出流重定向到某个日志驱动（Logging Driver),比如`/var/log/container`,(**容器自身的特性**)

每个node只需要以`DaemonSet`的方式运行pod，pod主要是对对本节点（node上的）/var/log和 /var/lib/docker/containers/的日志推送到远端日志服务器

**方案二的原理：**

以sidebar的方式在需要收集的pod起agent，直接发送到后端的日志存储
![](https://files.mdnice.com/user/4251/1d54ce33-a220-4ca4-b56c-2c5f0b388941.png)

**方案三不讨论**

**实际设计：**

方案只有这两种，要处理好用户的需求的话，并实现一定的自动化管理，最好的方式是提供给用户一个申请页面，后审批，审批通过后，日志采集sidebar自动部署到环境中，且用户能看到一个通用的`dashboard`，后续都自动完成。

**日志重要的是规范，通过统一的入口来实现规范**





