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