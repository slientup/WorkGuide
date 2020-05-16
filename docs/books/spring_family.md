# 极客时间 spring 全家桶 笔记

一 JDBC
----
#### 最好用的连接池 HikariCP
HikariCP的优点就是特别快，spring 2.0开始默认就是HikariCP的数据库连接池 详细介绍可参考HikariCP github主页
#### 最好用的连接池 Druid 
阿里巴巴开源的连接池Druid，非常全面的监控

#### 关于JPA、JDBC和mybatis
 JPA是规范，Hibernate才是实现，Hibernate和MyBatis 都能用来做数据的存取操作，它们底下是JDBC支持。我想你提问里说的应该是spring-data-jdbc，
 它的定位和Hibernate还有MyBatis是一样的，用来方便地操作数据。
 
 JPA和JDBC都是规范，JPA可以看做ORM的一套规范，Hibernate是JPA的实现；JDBC提供了Java数据库操作的底层规范，各种连接啊、查询啊什么的操作都是JDBC来
 定义的，不同数据库都提供了遵循JDBC的驱动;Druid和HikariCP都是数据源，或者简单点说是数据库连接池；ShardingShpere是用来做分库分表。平时这些东西是要根据实际情况结合在一起来使用
 
 大型项目中如果使用到分库分表的技术，建议使用`中间件`来单独的做分库分表，程序就不用关注分库分表相关的操作,如果只是做读写分离的话，可以在程序端进行修改。
 

#### 常用注解   
**Java Config 相关注解**  
- @Configuration
- @ImportResource
- @ComponentScan
- @Bean
- @ConfigurationProperties 
**定义相关注解**  
- @Component / @Repository 数据库访问层/ @Service
- @Controller / @RestController(ResposeBody+Controller)
- @RequestMapping

**注⼊相关注解   **优先使用Autowired注解 引入接口或者类有多个的时候，再使用Qualifier指定具体的名字**  
- @Autowired / @Qualifier（指定名字） / @Resource
- @Value（注入常量）
[@Autowired注解与@Qualifier注解搭配使用](https://www.cnblogs.com/hjw-zq/p/10626347.html)
    
#### 如何解禁 Endpoint
 /actuator/health 健康检查  
 /actuator/beans 查看容器中的所有 Bean  
 /actuator/mappings 查看 Web 的 URL 映射  
 /actuator/env 查看环境信息)  
 默认 actuator/health 和 /actuator/info 可 Web 访问   
 **解禁所有 Endpoint**   
 • application.properties / application.yml  
 • management.endpoints.web.exposure.include=*   

假设你的多租户是B端的商户，一般商户也会分小商户、中型商户和大商户，小商户大家可以共用一套表，中型的自己独占一套，大商户自己还需要分库分表。可以通过中间件配合路由表来实现。系统里只要保证每次操作数据库都带上商户号，剩下的交给中间件就好了

druid 支持slow query 日志 在配置文件中开启就可以

二 ORM
----
Hibernate 和 Mybatis 都是ORM的实现

JPA是规范，Hibernate是JPA的一种实现，Spring Data JPA（对hibernate又做了一层抽象）可以帮助大家更方便地使用JPA，它底层用了Hibernate。
Hibernate替我们做了ORM的工作，简单的场景中，你并不需要手写SQL。   

MyBatis跟JPA没什么关系，对于不怎么复杂的项目，我觉得JPA（也就是Hibernate）更简单点。MyBatis对底层SQL的把控度更强一点，更有利于DBA对SQL做优化。
所以大厂更偏向mybatis

源码学习 Spring Data JPA的Repository是怎么从接口变成Bean的
SimpleJpaRepository里面就定义了简单的sql查询方法

**让mybatis更好用的工具 MyBatis Generator**
该工具自动生成mapper xml model 等代码
项目中如果存在手写和生成的mapper，建议分开放置，分开放能够很好的支持后期的维护，而且自动生成的不建议再修改
自动生成的都是单表的CRUD，join的你只能靠自己

**让mybatis更好用的工具 MyBatis PageHelper**
顾名思义该工具目的就是用来做分页的插件

**让mybatis更好用的工具 MyBatis-Plus**

Spring Boot针对数据源可以做初始化，默认是针对内嵌数据源**自动执行schema.sql和data.sql**，这个也可以配置为always，一直执行。spring.datasource.initialization-mode=always

Spring Boot + H2 database数据库

三 NoSQL
----
docker 的好处 在开发的人眼中 可快速搭建开发中工具 如mongodb redis
windows 安装docker 操作步骤 https://mp.weixin.qq.com/s?__biz=Mzg3MzAyODY2Nw==&mid=100000009&idx=1&sn=bcb5680973c15835be0abd50f4d290ff&chksm=4ee70c5d7990854bdcfca932d05aab2ba17acec4ef3224e29a593fcdcadd1981154638c3fad4#rd

#### mongodb的操作
支持两种方式
- `Template`  能做各式各样的操作，比较灵活，但什么都得自己写 累啊 
- `Repository` 则更符合DDD里数据仓库的习惯，做了很好的封装，用起来比较方便，更符合我们的操作系统 但非常复杂的需要用Template来实现
repository相对封装地更多一些，用起来更方便，xxxTemplate这套给你更大的灵活度，如果是普通的场景里用用repository就够了

mysql和mongodb各有适用领域，mongodb查询效率高，更适合海量数据

#### redis的操作
spring data redis
- 支持客户端Jedis / lettuce
- redisTemplate
- Repository支持

#### Spring的缓存抽象  这是spring平台为我们提供的
缓存的另一种实现方式:Spring的缓存抽象  通过这种方式也可以实现缓存，而不用redis,支持本地缓存，分布式redis缓存
spring的缓存抽象默认情况下使用的jvm来做缓存的(本地缓存)，也可以指定通过redis来做
至于spring.cache.cache-names=coffee，是用来配置默认缓存的，启动时会把其中指定的缓存创建出来。@CacheConfig用来配置类级别共享的缓存配置

什么样的数据使用什么样的缓存？
- 对于经常不变的数据，一天都可能不变的这样，又没有分布式的需求，则使用本地缓存
- 对于经常读的高频数据，且读远大于写的场景，又有分布式的需求，则使用redis缓存
- 对于读并不频繁的数据 直接扔数据库里面就行了。

设置最大的重定向次数的，RedisCluster里，你把请求发到了某个节点上，它发现这个KEY不在自己这里，就会告诉你重定向到另一个节点上去找。
完全用Jedis或者Lettuce客户端什么都做，也就是最灵活的。但如果你就想缓存一些方法的执行结果，Spring的缓存抽象都帮你写好了，直接用就行
Spring的缓存抽象是将方法的执行结果缓存起来，而客户端或者redisTemplate可以缓存任意字段

四 Reactor 
----
响应式编程参考链接: https://www.ibm.com/developerworks/cn/java/j-cn-with-reactor-response-encode/index.html
响应式编程 bhttps://zhuanlan.zhihu.com/p/45351651  

响应式编程通过`推送的模式`，rabbitmq消息队列中的推送方式来实现自动更新，而不是`拉的方式`，一种异步编写方案
消息队列的实现借助于第三方中间件产品，而响应式编程自己来实现；

反应式编程来源于数据流和变化的传播，意味着由底层的执行模型负责通过数据流来自动传播变化。比如求值一个简单的表达式 c=a+b，当 a 或者 b 的值发生变化时，传统的编程范式需要对 a+b 进行重新计算来得到 c 的值。如果使用反应式编程，当 a 或者 b 的值发生变化时，c 的值会自动更新。反应式编程最早由 .NET 平台上的 Reactive Extensions (Rx) 库来实现。后来迁移到 Java 平台之后就产生了著名的 RxJava 库，并产生了很多其他编程语言上的对应实现。在这些实现的基础上产生了后来的反应式流（Reactive Streams）规范。该规范定义了反应式流的相关接口，并将集成到 Java 9 中。

在传统的编程范式中，我们一般通过迭代器（Iterator）模式来遍历一个序列。这种遍历方式是由调用者来控制节奏的，采用的是拉的方式。每次由调用者通过 next()方法来获取序列中的下一个值。使用反应式流时采用的则是推的方式，即常见的发布者-订阅者模式。当发布者有新的数据产生时，这些数据会被推送到订阅者来进行处理。在反应式流上可以添加各种不同的操作来对数据进行处理，形成数据处理链。这个以声明式的方式添加的处理链只在订阅者进行订阅操作时才会真正执行。

反应式流中第一个重要概念是负压（backpressure）。在基本的消息推送模式中，当消息发布者产生数据的速度过快时，会使得消息订阅者的处理速度无法跟上产生的速度，从而给订阅者造成很大的压力。当压力过大时，有可能造成订阅者本身的奔溃，所产生的级联效应甚至可能造成整个系统的瘫痪。负压的作用在于提供一种从订阅者到生产者的反馈渠道。订阅者可以通过 request()方法来声明其一次所能处理的消息数量，而生产者就只会产生相应数量的消息，直到下一次 request()方法调用。这实际上变成了`推拉结合`的模式。

五 spring mvc
----
要容器造什么对象是由我们自己决定的,决定对象间依赖关系的也是我们自己,容器只是提供管理对象的空间而已.那么问题就出现了,
**我们怎么向容器中放入我们需要的对象呢?**
在spring中就是由**ApplicationContext应用上下文**来实现  将你需spring帮你管理的对象放入容器的一种对象.可以理解为spring容器抽象的一种实现

我们常见的ApplicationContext本质上说就是一个维护Bean定义以及对象之间协作关系的高级接口  还记得最开始写spring框架的时候用的上下文加载bean

idea中集成的http请求的插件：`RestfulToolkit`
可以直接debug进依赖的库里的，就用你自己的工程就好 然后通过网页访问某个接口的方式直接debug

先在依赖包找到对应的DispatcherServlet 打开，Intellj会在页面提示`Download Source`下载 如果不下载的话，就只能看到class文件，class文件是看不到注解的，打上断点然后debug的方式运行程序 然后就可以进行调试运行了


### spring Interceptor
拦截器与spring mvc处理流程有关系，我们实际写的`controller`处于流程的中间，可看源代码`doservice()`    
1. 在主应用程序中 add 自定义`PerformanceInteceptor`
```
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(new PerformanceInteceptor())
				.addPathPatterns("/coffee/**").addPathPatterns("/order/**");
	}
```
2.配置拦截器  实现统计时间的功能
```
public class PerformanceInteceptor implements HandlerInterceptor {
    private ThreadLocal<StopWatch> stopWatch = new ThreadLocal<>();

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        StopWatch sw = new StopWatch();
        stopWatch.set(sw);
        sw.start();
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        stopWatch.get().stop();
        stopWatch.get().start();
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        StopWatch sw = stopWatch.get();
        sw.stop();
        String method = handler.getClass().getSimpleName();
        if (handler instanceof HandlerMethod) {
            String beanType = ((HandlerMethod) handler).getBeanType().getName();
            String methodName = ((HandlerMethod) handler).getMethod().getName();
            method = beanType + "." + methodName;
        }
        log.info("{};{};{};{};{}ms;{}ms;{}ms", request.getRequestURI(), method,
                response.getStatus(), ex == null ? "-" : ex.getClass().getSimpleName(),
                sw.getTotalTimeMillis(), sw.getTotalTimeMillis() - sw.getLastTaskTimeMillis(),
                sw.getLastTaskTimeMillis());
        stopWatch.remove();
    }
}
```

七 访问web资源
--- 
restTemplate spring boot未直接提供实例 我们需要通过bean的方式先将restTemplate注入容器

restTemplate是spring对底层http请求的封装，`ClientHttpRequestFactory`是通过接口，默认使用`SimpleClientHttpRequestFactory`来实现
实现http的请求，该类实际调用的是jdk的`HttpURLConnection`，但在工作中，我们多半会修改底层实现的类库，该接口也支持`HttpClient`， `OKhttp`,'webclient'

我们定制后台请求模板，主要优化的是请求策略   
`connectTimeout` `readTimeout`  `连接池`等信息

简单例子：
注意点就是，如果请求返回的结果直接序列化成对应的对象实例的话，则该对象必须实现`implements Serializable` 我们在controller层接收的类不用实现序列化
是因为`requestbody`这个注解，会帮我们序列化


```
String coffeeUri = "http://localhost:8080/coffee/";
Coffee request = Coffee.builder()
		.name("Americano")
		.price(Money.of(CurrencyUnit.of("CNY"), 25.00))
                .build();
		
Coffee response = restTemplate.postForObject(coffeeUri, request, Coffee.class);  

log.info("New Coffee: {}", response);

// 这里coffee 类的定义

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Coffee implements Serializable {   //这里必须继承序列化接口
    private Long id;
    private String name;
    private Money price;
    private Date createTime;
    private Date updateTime;
}

```

可通过官网查看具体定制为httpclient的操作

webclient  相当于异步请求

创建对象实例 create() build()   
获取结果 exchange()  retrieve()


八  Web开发进阶
----
restful 相关的论文 https://www.infoq.cn/article/web-based-apps-archit-design





