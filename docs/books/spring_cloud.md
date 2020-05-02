# Spring cloud 微服务实战

一 基础知识
----
#### 什么是微服务架构？
微服务是系统架构上的的一种设计风格，它的主旨是将一个**原本独立的系统**拆分成很多小型服务，这些小型服务都在各自独立
的进程中运行，服务之间通过基于HTTP的RESTful API进行通信。

微服务架构中，通常会使用以下两种服务调用方式：
- 使用HTPP的RESTful API或者轻量级的消息发送协议，实现信息传递与服务调用的触发，
- 通过rabbitmq等提供可靠的异步交换的中间件


spring cloud 提供的是解决微服务架构实施的综合性解决框架。分布式事务本身的实现难度就非常大，所以在微服务的架构中
，我们更多的强调在各服务之间进行"无事务"的调用，而对于数据一致性，只要求数据在最后处理状态是一致的即可，若过程发现
错误，通过补偿机制来进行处理，使得错误数据最终能达到一致性。

二 微服务构建：spring boot
---- 
#### 多环境配置
多环境的配置文件名需要满足`application-{profile}.properties`格式，例如
- application-test.properties:测试环境
- application-prod.properties:生产环境
具体哪个配置文件被加载，需要在`application.properties`文件中通过`spring.profiles.active`属性值来决定，如`spring.profiles.active=test`就会
加在测试环境的配置

#### 配置属性加载顺序
1. 命令行中输入的参数
2. `SPRING_APPLICATTION_JSON`中的属性
3. java:comp/env中的JNDI属性
4. java的系统属性，可以通过System.getProperties()获得的内容
5. 操作系统的环境变量
6. 通过random.*配置的随机属性
7. **位于当前应用jar包之外**，针对不同的`{profile}`环境的配置文件内容，例如`application-{profile}.properties` 或是Yaml定义的配置文件
8. **位于当前应用jar包之内**，针对不同的`{profile}`环境的配置文件内容，例如`application-{profile}.properties` 或是Yaml定义的配置文件
9. **位于当前应用jar包之外**`的application.properties` 或是Yaml定义的配置文件
10. **位于当前应用jar包之内**`的application.properties` 或是Yaml定义的配置文件 
优先级按上面的顺序由高到低，数字越小`优先级越高`。而远端配置管理中心就是利用7和9来实现的

#### acuator监控模块
spring boot应用中监控应用性能非常全面的模块 提供了各类api接口来查看当前运行的状态；
- /autoconfig
- /beans
- /configprops
- /env
- /mappings
- /metrics
- /health
- /dump
- /trace







