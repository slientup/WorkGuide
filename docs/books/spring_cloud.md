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


API 网关服务：spring cloud zuul
---- 
API 网关服务是分布式系统对外提供服务的统一入口，而这个统一入口可以做很多事情，与实际业务无关的事情都可以交给网关来做，比如**验证授权限流负载均衡等**

#### 路由中的路径匹配
通配符   说明
？       匹配任意单个字符    /user-service/?   匹配 /user-servcie/a  /user-servcie/b  但不匹配/user-servcie/ab
*        匹配任意数量的字符  /user-service/*   匹配 /user-servcie/ab  /user-servcie/abc  但不匹配/user-servcie/a/b
**       匹配任意数量的字符 支持多级目录 /user-service/** 除了匹配上面的，还能匹配/user-servcie/a/b

#### 服务路由规则
```
zuul.routes.service-order=/order/**
zuul.routes.service-order.serviceId=order
```
对于这种规则性的配置默认zuul已经配置好

#### Hystrix和Ribbon支持
Zuul默认就对这两个特性支持，前提是使用的`path和serviceId`的组合来进行配置

#### 过滤器详解
zuul主要提供两个功能路由和过滤，而实际在实现中都是通过过滤器来实现，zuul提供一个很长的过滤器的链路，而我们可以通过对过滤器自定义来实现pre或者post后的处理逻辑；
过滤器主要由四部分构成，过滤类型，执行顺序，执行条件，具体操作 
```
String filterType();
int filterOrder();   通过int值来定义过滤器的执行顺序
boolean shouldFilter();  返回一个boolean类型来判断该过滤器是否要执行
Object run();   具体执行内容
```
过滤类型决定了执行的阶段：
- `pre`：可以在请求被路由之前调用
- `route`：在路由请求时候被调用
- `post`：在route和error过滤器之后被调用
- `error`：处理请求时发生错误时被调用

#### 请求生命周期
![请求生命周期](https://github.com/slientup/WorkGuide/blob/master/filter.png)

从上图中，我们可以看到，当外部HTTP请求到达API网关服务的时候，首先它会进入第一个阶段pre，在这里它会被pre类型的过滤器进行处理，该类型的过滤器主要目的是在进行请求路由之前做一些前置加工，比如请求的校验等。在完成了pre类型的过滤器处理之后，请求进入第二个阶段routing，也就是之前说的路由请求转发阶段，请求将会被routing类型过滤器处理，这里的具体处理内容就是将外部请求转发到具体服务实例上去的过程，当服务实例将请求结果都返回之后，routing阶段完成，请求进入第三个阶段post，此时请求将会被post类型的过滤器进行处理，这些过滤器在处理的时候不仅可以获取到请求信息，还能获取到服务实例的返回信息，所以在post类型的过滤器中，我们可以对处理结果进行一些加工或转换等内容。另外，还有一个特殊的阶段error，该阶段只有在上述三个阶段中发生异常的时候才会触发，但是它的最后流向还是post类型的过滤器，因为它需要通过post过滤器将最终结果返回给请求客户端

#### 核心过滤器
核心过滤器是zuul内置的过滤器，当某个请求到达该zuul的时候就会经过这些核心过滤器链和自定义的过滤器
![核心过滤器](http://blog.didispace.com/assets/zuul-filter-core.png)



 消息流 spring cloud stream
---- 
spring cloud stream组件就是解决跟消息队列等中间件打交道，如kafka、rabbitmq，spring cloud stream对其做了一层封装，使我们能方便与中间件通信


 分布式服务链路跟踪 spring cloud sleuth
---- 
在分布式系统中，前端任何一个操作都会形成一条复杂的调用链路，这条链路过程中会出现依赖的某个服务延迟过高或者错误，而**全链路跟踪sleuth就是跟踪一条
请求的调用链路，以及对应的延时信息**

跟踪链路主要有如下四个字段        
- 当前应用名称  
- 当前请求链路id  **在一次服务请求链中会保持并传递同一个traceid**  
- 基本的工作单元id 可用于统计延时信息  
- 布尔型 是否输出到zipkin等外部收集器中  

日志的格式为：[application name, traceId, spanId, export]      
`application name` — 应用的名称，也就是application.properties中的spring.application.name参数配置的属性。  
`traceId` — 为一个请求分配的ID号，用来标识一条请求链路。  
`spanId` — 表示一个基本的工作单元，一个请求可以包含多个步骤，每个步骤都拥有自己的spanId。一个请求包含一个TraceId，多个SpanId  
`export` — 布尔类型。表示是否要将该信息输出到类似Zipkin这样的聚合器进行收集和展示。   

在日志输出的时候增加这四个字段信息，日志举例如下:
2018-04-16 16:35:55.161  INFO [**Sleuth Tutorial,0afe3e67168fce4f,0afe3e67168fce4f,false**] 5968 --- [nio-8080-execc.p.spring.cloud.sleuth.SleuthService    : Doing some work

**跟踪原理：**

为了实现请求跟踪，当请求发送到分布式系统的入口端点时，只需要服务跟踪框架为该请求创建一个唯一的跟踪标识，**同时在分布式系统内部流转的时候，框架始终保持传递该唯一标识，直到返回给请求方为止**，这个唯一标识就是前文中提到的Trace ID。通过Trace ID的记录，我们就能将所有请求过程日志关联起来。

为了统计各处理单元的时间延迟，当请求达到各个服务组件时，或是处理逻辑到达某个状态时，也通过一个唯一标识来标记它的开始、具体过程以及结束，该标识就是我们前文中提到的Span ID，对于每个Span来说，**它必须有开始和结束两个节点，通过记录开始Span和结束Span的时间戳，就能统计出该Span的时间延迟**，除了时间戳记录之外，它还可以包含一些其他元数据，比如：事件名称、请求信息等

**与logstash整合**
spring boot 与elk对接的时候，可以只与logstash对接就行，在配置文件中增加对logstash的`appender`就可以

**与Zipkin整合**

能提供对各个阶段的延时关注，更细致







