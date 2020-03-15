
- [产生背景](#产生背景)

- [Apollo](#Apollo)
    - [架构](#架构)
    - [客户端案例](#客户端使用案例)
    - [源码分析](#源码分析)
- [spring-cloud-config](#spring-cloud-config)
    - [架构](#架构)
- [参考链接](#参考链接)
    
### 产生背景  
**困境**：在写单机应用的时候，我们通常将跟配置有关的常量放入到配置文件中，它会带来以下问题，我要做任何一个参数的修改都要重启服务，甚至重新打包，也许单机应用你可以忍受，但是在分布式场景，几十台vm甚至几百台的时候，你就无法忍受每台设备去更改，还要重启服务。为了解决这个问题，你可能尝试将配置参数配置在数据库中，每次程序调用时都去拉取一遍，这对第三方可靠性运行就要求很大了，而且这些都是没必要存在的流量，随着发展对配置要求越来越高，实时生效，灰度发布，分环境、分集群管理配置，完善的权限、审核机制....
**分布式配置中心目标**:将配置文件从应用程序中脱离出来，**统一管理，统一下发，灰度发布，实时生效，不需要重启服务。**   
**技术思路**: 单一的配置中心，能感知到配置是否变化，一旦变化通知订阅的用户(监听器的方式注册)，告知对方重新拉取最新的配置  
**相关产品**: 百度`disconf`  携程`Apollo`   `spring-cloud-config`


### Apollo
#### 架构
##### 基础版
 ![基本架构](https://github.com/ctripcorp/apollo/raw/master/doc/images/basic-architecture.png)
1. 用户在配置中心对配置修改并发布  
2. 配置中心通知Apollo客户端有配置更新  
3. Apollo客户端从配置中心拉取最新的配置、更新本地配置并通知应用   
##### 增强版
![增强版](https://github.com/ctripcorp/apollo/raw/master/doc/images/client-architecture.png)
**增加定时更新功能**  
上图简要描述了Apollo客户端的实现原理：  
1. 客户端和服务端保持了一个长连接，从而能第一时间获得配置更新的推送。（通过Http Long Polling实现）
2. 客户端还会定时从Apollo配置中心服务端拉取应用的最新配置。
   - 这是一个`fallback`机制，为了**防止推送机制失效导致配置不更新**
   - 客户端定时拉取会上报本地版本，所以一般情况下，对于`定时拉取`的操作，服务端都会返回`304 - Not Modified`
   - 定时频率默认为每`5分钟`拉取一次，客户端也可以通过在运行时指定`System Property: apollo.refreshInterval`来覆盖，单位为分钟。
3. 客户端从Apollo配置中心服务端获取到应用的最新配置后，会保存在内存中
4. 客户端会把从服务端获取到的配置在本地文件系统缓存一份
   - 在遇到服务不可用，或网络不通的时候，**依然能从本地恢复配置**
5. 应用程序可以从Apollo客户端获取最新的配置、订阅配置更新通知
##### 完整架构演进版
- [微服务架构~携程Apollo配置中心架构剖析](https://mp.weixin.qq.com/s/-hUaQPzfsl9Lm3IqQW3VDQ)
架构演进路径：
1. 为了保证**高可用**实现服务注册发现功能，引进了`spring cloud Eureka`，Config/AdminService启动后都会注册到`Eureka`服务注册中心，并定期发送保活心跳 
2. 但Eureka的客户端并不支持所有语言，默认只支持java，为了让其他语言的客户端也能接入进来，引入了`MetaServer`这个角色，它其实是一个`Eureka`的`Proxy`,提供一个api接口，后台调用Eureka，client 通过它来发现服务提供者；  
3. 为保证`MetaServer`的可靠性，部署多台，client通过域名的方式访问到slb再到最终的metaserver;  
 ![完整架构](https://mmbiz.qpic.cn/mmbiz_png/ELH62gpbFmGdnIjxDT7AOQyZgl2KQnz6LCwSGeZjrh5DlMd0MMxVIepCFQKdE6vfJWbZOKiaHqEcmia1nJia2o7Vg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 客户端使用案例
Apollo支持`API`和`spring`整合的方式接入  
1. API的方式功能完备，可配置**值实时**更新(热发布)
2. Spring接入方式，与本地配置文件使用一样
    - Placeholder方式 代码中直接使用，如：`@Value("${someKeyFromApollo:someDefaultValue}")`
    - Spring boot的`@ConfigurationProperties`方式
    - 从v0.10.0开始的版本支持placeholder在`运行时自动更新`
3. [代码使用场景](https://github.com/ctripcorp/apollo-use-cases)
##### API使用方式
- 获取配置属的值：

        Config config = ConfigService.getAppConfig(); //config instance is singleton for each namespace and is never null
        String someKey = "someKeyFromDefaultNamespace";
        String someDefaultValue = "someDefaultValueForTheKey";
        String value = config.getProperty(someKey, someDefaultValue);
- 配置监听事件（需要热发布的）

        Config config = ConfigService.getAppConfig(); //config instance is singleton for each namespace and is never null
        config.addChangeListener(new ConfigChangeListener() {
            @Override
            public void onChange(ConfigChangeEvent changeEvent) {
                System.out.println("Changes for namespace " + changeEvent.getNamespace());
                for (String key : changeEvent.changedKeys()) {
                    ConfigChange change = changeEvent.getChange(key);
                    System.out.println(String.format("Found change - key: %s, oldValue: %s, newValue: %s, changeType: %s",   change.getPropertyName(), change.getOldValue(), change.getNewValue(), change.getChangeType()));
                }
            }
        });
 
##### spring boot 使用方式
EnableApolloConfig 注解很关键 该注解会注入实例，连接到apolloconfig的服务器上，配置监听器  
注意`@EnableApolloConfig`要和`@Configuration`一起使用，不然不会生效
       
           //这个是最简单的配置形式，一般应用用这种形式就可以了，用来指示Apollo注入application namespace的配置到Spring环境中
            @Configuration
            @EnableApolloConfig
            public class AppConfig {
              @Bean
              public TestJavaConfigBean javaConfigBean() {
                return new TestJavaConfigBean();
              }
            }

   
#### 源码分析


### 参考链接
- [apollo配置中心介绍](https://github.com/ctripcorp/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E4%BB%8B%E7%BB%8D)
- [apollo java客户端使用介绍](https://github.com/ctripcorp/apollo/wiki/Java%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97)
- [携程Apollo配置中心架构剖析](https://mp.weixin.qq.com/s/-hUaQPzfsl9Lm3IqQW3VDQ)








