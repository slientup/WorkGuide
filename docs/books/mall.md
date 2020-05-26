# 开源项目mall 源码阅读笔记
* [mall-common](#mall-common)
* [mall-admin](#mall-admin)
* [mall-security](#mall-security)
* [AOP](#AOP)
* [跨域问题](#跨域问题)
* [mall-admin-web](#mall-admin-web)




### mall-common 
该模块存放其他模块都可能调用的类 主要是对`api`结果的封装，`exception` 异常的封装，统一管理   

#### `exception`模块
定义了Assert类，其它类调用该类抛出异常
```

public class Asserts {
    public static void fail(String message) {
        throw new ApiException(message);
    }

    public static void fail(IErrorCode errorCode) {
        throw new ApiException(errorCode);
    }
}
```
再定义了`GlobalExceptionHandler`类，其它类调用该类抛出异常 该类专门捕获`ApiException`异常，并对其进行处理
`@ResponseBody` 这个注解**一定要添加** 这个注解是会将return的结果返回给`api`的调用方
```
@ControllerAdvice
public class GlobalExceptionHandler {

    @ResponseBody   // 这个注解必须要 没有它程序不会返回给用户
    @ExceptionHandler(value = ApiException.class)
    public CommonResult handle(ApiException e) {
        if (e.getErrorCode() != null) {
            return CommonResult.failed(e.getErrorCode());
        }
        return CommonResult.failed(e.getMessage());
    }
}
```
`ApiException`异常类的定义
```
public class ApiException extends RuntimeException {
    private IErrorCode errorCode;

    public ApiException(IErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }
```
在service层调用断言`Asserts.fail`
```
        if (!hasStock(cartPromotionItemList)) {
            Asserts.fail("库存不足，无法下单");
        }
```
小知识点 在所有涉及到订单类的处理逻辑方法，都设计成`事务`的方式  且在**接口**进行声明
```
public interface OmsPortalOrderService {
    /**
     * 根据用户购物车信息生成确认单信息
     */
    ConfirmOrderResult generateConfirmOrder();

    /**
     * 根据提交信息生成订单
     */
    @Transactional
    Map<String, Object> generateOrder(OrderParam orderParam);
```
#### `api`模块
api接口模块是对**所有返回结果**进行的封装，增加`status`和`message`字段(状态解释)  提供给用户更多有用信息

在设计该类的时候，由于这是通用的类，并不知道传入进来的data的类型，所以该类是`模板类`
```
public class CommonResult<T> {
    private long code;
    private String message;
    private T data;

    protected CommonResult() {
    }

    protected CommonResult(long code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    /**
     * 成功返回结果
     *
     * @param data 获取的数据
     */
    public static <T> CommonResult<T> success(T data) {
        return new CommonResult<T>(ResultCode.SUCCESS.getCode(), ResultCode.SUCCESS.getMessage(), data);
    }
    
```
`枚举类`的定义值得借鉴
```
public enum ResultCode implements IErrorCode {
    SUCCESS(200, "操作成功"),
    FAILED(500, "操作失败"),
    VALIDATE_FAILED(404, "参数检验失败"),
    UNAUTHORIZED(401, "暂未登录或token已经过期"),
    FORBIDDEN(403, "没有相关权限");

    private long code;
    private String message;

    private ResultCode(long code, String message) {
        this.code = code;
        this.message = message;
    }

    public long getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}

```
controller层对`CommonResult.success`的调用 
```
    @ApiOperation(value = "根据专题名称分页获取专题")
    @RequestMapping(value = "/list", method = RequestMethod.GET)
    @ResponseBody
    public CommonResult<CommonPage<CmsSubject>> getList(@RequestParam(value = "keyword", required = false) String keyword,
                                                        @RequestParam(value = "pageNum", defaultValue = "1") Integer pageNum,
                                                        @RequestParam(value = "pageSize", defaultValue = "5") Integer pageSize) {
        List<CmsSubject> subjectList = subjectService.list(keyword, pageNum, pageSize);
        return CommonResult.success(CommonPage.restPage(subjectList));
    }
```

### mall-admin
后台管理系统
#### resource目录结构
![resource目录结构](https://github.com/slientup/WorkGuide/blob/master/docs/books/mall_admin_resource.png)
- dao  数据库操作命令文件
   - *.xml     
- application.yml    配置文件
- application-dev.yml  开发环境的配置文件
- application-prod.yml  生产环境的配置文件
- logback-spring.xml  日志文件 

**dao**
mybatis访问数据库有两种方式实现，一种基于`xml`，另一种基于`注解`(放在`service`层) 这里选择的是基于`xml`的方式，他们的唯一区别
是把sql语句放在`注解`里，还是`xml`中.
mybatis的应用中分层架构  `controller`--->`service`---->`dao(mappper)`--访问数据库操作   
dao(mapper)----数据库bean----xml  这三个文件都是用来对数据库的操作，对上层service来说 提供的就是某个方法，即对哪个数据库进行什么操作
由service来调用，dao以下的进行封装   
自定义的数据库操作 `dao`层  生成器生成的我们定义为`mapper`层  生成器是根据数据库生成的，前提是先创建数据库里面的表

`@MapperScan`注解：用于扫描对应包里面的mapper进入spring boot容器中，方便`@Autowired`使用   
```
@Configuration
@EnableTransactionManagement
@MapperScan({"com.macro.mall.mapper","com.macro.mall.dao"})   统一引入
public class MyBatisConfig {
}
```
参考资料:[springboot_mybatis](https://github.com/Snailclimb/springboot-guide/blob/master/docs/basis/springboot-mybatis.md)    


**application.yml**
该配置文件存放的`prod`和`dev`环境共有的配置内容 主要包含几大块`spring` `mysbatis` `jwt` `redis` `secure` `aliyun` `log`
```
spring:
  profiles:
    active: dev #默认为开发环境
  servlet:
    multipart:
      enabled: true #开启文件上传
      max-file-size: 10MB #限制文件上传大小为10M

mybatis:
  mapper-locations:
    - classpath:dao/*.xml
    - classpath*:com/**/mapper/*.xml

jwt:
  tokenHeader: Authorization #JWT存储的请求头
  secret: mall-admin-secret #JWT加解密使用的密钥
  expiration: 604800 #JWT的超期限时间(60*60*24*7)
  tokenHead: Bearer  #JWT负载中拿到开头

redis:
  database: mall
  key:
    admin: 'ums:admin'
    resourceList: 'ums:resourceList'
  expire:
    common: 86400 # 24小时


secure:
  ignored:
    urls: #安全路径白名单
      - /swagger-ui.html
      - /swagger-resources/**
      - /swagger/**
      - /**/v2/api-docs
      - /**/*.js
      - /**/*.css
      - /**/*.png
      - /**/*.ico
      - /webjars/springfox-swagger-ui/**
      - /actuator/**
      - /druid/**
      - /admin/login
      - /admin/register
      - /admin/info
      - /admin/logout
      - /minio/upload

aliyun:
  oss:
    endpoint: oss-cn-shenzhen.aliyuncs.com # oss对外服务的访问域名
    accessKeyId: test # 访问身份验证中用到用户标识
    accessKeySecret: test # 用户用于加密签名字符串和oss用来验证签名字符串的密钥
    bucketName: macro-oss # oss的存储空间
    policy:
      expire: 300 # 签名有效期(S)
    maxSize: 10 # 上传文件大小(M)
    callback: http://39.98.190.128:8080/aliyun/oss/callback # 文件上传成功后的回调地址
    dir:
      prefix: mall/images/ # 上传文件夹路径前缀

minio:
  endpoint: http://192.168.3.101:9090 #MinIO服务所在地址
  bucketName: mall #存储桶名称
  accessKey: minioadmin #访问的key
  secretKey: minioadmin #访问的秘钥

logging:
  level:
    root: info #日志配置DEBUG,INFO,WARN,ERROR
    com.macro.mall: debug
#  file: demo_log.log #配置日志生成路径
#  path: /var/logs #配置日志文件名称

```
**application-prod.yml**
生产环境配置 主要包含 数据库和日志的配置  数据库连接池使用阿里的`druid`
```
spring:
  datasource:
    url: jdbc:mysql://db:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: reader
    password: 123456
    druid:
      initial-size: 5 #连接池初始化大小
      min-idle: 10 #最小空闲连接数
      max-active: 20 #最大连接数
      web-stat-filter:
        exclusions: "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*" #不统计这些请求数据
      stat-view-servlet: #访问监控网页的登录用户名和密码
        login-username: druid
        login-password: druid
  redis:
    host: redis # Redis服务器地址
    database: 0 # Redis数据库索引（默认为0）
    port: 6379 # Redis服务器连接端口
    password: # Redis服务器连接密码（默认为空）
    timeout: 300ms # 连接超时时间（毫秒）
    
logging:
  path: /var/logs #配置日志生成路径
```
**logback.xml** 
日志同时记录到文件和logstash中，然后logstash讲日志信息发送到es    
参考资料：[SpringBoot应用整合ELK实现日志收集](http://www.macrozheng.com/#/technology/mall_tiny_elk)
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
    <!--应用名称-->
    <property name="APP_NAME" value="mall-admin"/>
    <!--日志文件保存路径-->
    <property name="LOG_FILE_PATH" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/logs}"/>
    <contextName>${APP_NAME}</contextName>
    <!--每天记录日志到文件appender-->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/${APP_NAME}-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
    </appender>
    <!--输出到logstash的appender-->
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:4560</destination>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="LOGSTASH"/>
    </root>
</configuration>
```


#### Validator
自定义 `Validator`(通过注解的方式)  通过实现`ConstraintValidator`接口来实现校验器

定义 `FlagValidator`注解
```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD,ElementType.PARAMETER})
@Constraint(validatedBy = FlagValidatorClass.class)    // 指定了实现的该接口实现的class
public @interface FlagValidator {
    String[] value() default {};

    String message() default "flag is not found";   //  校验器未通过返回的信息
    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```
定义`FlagValidatorClass`实现`ConstraintValidator<FlagValidator,Integer>`接口
```
public class FlagValidatorClass implements ConstraintValidator<FlagValidator,Integer> {
    private String[] values;
    @Override
    public void initialize(FlagValidator flagValidator) {
        this.values = flagValidator.value();    // 初始化
    }

    @Override
    public boolean isValid(Integer value, ConstraintValidatorContext constraintValidatorContext) {
        boolean isValid = false;
        if(value==null){
            //当状态为空时使用默认值
            return true;
        }
        for(int i=0;i<values.length;i++){
            if(values[i].equals(String.valueOf(value))){
                isValid = true;
                break;
            }
        }
        return isValid;
    }
}
```
参考资料：
[spring自定义vaildation](https://snailclimb.gitee.io/springboot-guide/#/./docs/advanced/spring-bean-validation)



### mall-security
该模块主要包含安全认证相关的配置主要使用`Spring Security`+`JWT`的技术 

**思路：**
在对用户进行认证和授权时，可以通过注解的方式在需要进行授权的时候，进行授权，但这种方式很繁琐，认证授权最好的方式就是通过过滤器在用户访问
真正的页面之前做统一处理。

Spring Security 相关接口和类
- `WebSecurityConfigurerAdapter`   安全相关的配置类 可以写类继承该类 核心
- `SecurityContextHolder` SecurityContextHolder存储安全上下文（security context）的信息。当前操作的用户是谁，该用户是否已经被认证，他拥有哪些角色权等等，这些都被保存在SecurityContextHolder中
- `UserDetails接口`  提供认证相关的用户的信息. 其主要的方法就是：String `getPassword()`; 和 String `getUsername()`;
- `User 类`  特指 org.springframework.security.core.userdetails 包中的 User 类。 它实现了 UserDetails 接口
- `UserDetailsService 接口` 作用是在特定用户权限认证时，用于加载用户信息。 该接口只有一个方法，用于返回用户的信息：UserDetails
  `loadUserByUsername`(String username) throws UsernameNotFoundException;那么，它的框架里面默认的实现类有 InMemoryUserDetailsManager，
   CachingUserDetailsService 和 JdbcDaoImpl，一个用于从内存中拿到用户信息，一个用于从数据库中拿到用户信息
常用注解
- `@EnableWebSecurity`  开启Spring Security的功能
- `@EnableGlobalMethodSecurity(prePostEnabled = true)` 可以开启security的注解，我们可以在需要控制权限的方法上面使用@PreAuthorize，@PreFilter这些注解

`WebSecurityConfigurerAdapter`的配置  这是整个`spring security`处理逻辑的**核心核心核心**
```
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
//使用@EnableGlobalMethodSecurity(prePostEnabled = true)
// 这个注解，可以开启security的注解，我们可以在需要控制权限的方法上面使用@PreAuthorize，@PreFilter这些注解。
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    //@Autowired
    //LightSwordUserDetailService lightSwordUserDetailService;

    @Override
    @Bean
    public UserDetailsService userDetailsService() { //覆盖写userDetailsService方法 (1)
        return new LightSwordUserDetailService();

    }

    /**
     * If subclassed this will potentially override subclass configure(HttpSecurity)
     *
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //super.configure(http);
        http.csrf().disable();

        http.authorizeRequests()
            .antMatchers("/").permitAll()
            .antMatchers("/amchart/**",
                "/bootstrap/**",
                "/build/**",
                "/css/**",
                "/dist/**",
                "/documentation/**",
                "/fonts/**",
                "/js/**",
                "/pages/**",
                "/plugins/**"
            ).permitAll() //默认不拦截静态资源的url pattern （2）
            .anyRequest().authenticated().and()
            .formLogin().loginPage("/login")// 登录url请求路径 (3)
            .defaultSuccessUrl("/httpapi").permitAll().and() // 登录成功跳转路径url(4)
            .logout().permitAll();

        http.logout().logoutSuccessUrl("/"); // 退出默认跳转页面 (5)

    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //auth
        //    .inMemoryAuthentication()
        //    .withUser("root")
        //    .password("root")
        //    .roles("ADMIN", "USER")
        //    .and()
        //    .withUser("admin").password("admin")
        //    .roles("ADMIN", "USER")
        //    .and()
        //    .withUser("user").password("user")
        //    .roles("USER");

        //AuthenticationManager使用我们的 lightSwordUserDetailService 来获取用户信息
        auth.userDetailsService(userDetailsService()); // （6）
    }

}

```
**mall项目的安全配置**  
```
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry registry = httpSecurity
                .authorizeRequests();
        //不需要保护的资源路径允许访问
        for (String url : ignoreUrlsConfig().getUrls()) {
            registry.antMatchers(url).permitAll();
        }
        //允许跨域请求的OPTIONS请求
        registry.antMatchers(HttpMethod.OPTIONS)
                .permitAll();
        // 任何请求需要身份认证   
        registry.and()
                .authorizeRequests()
                .anyRequest()
                .authenticated()
                // 关闭跨站请求防护及不使用session
                .and()
                .csrf()
                .disable()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                // 自定义权限拒绝处理类
                .and()
                .exceptionHandling()
                .accessDeniedHandler(restfulAccessDeniedHandler())
                .authenticationEntryPoint(restAuthenticationEntryPoint())
                // 自定义权限拦截器JWT过滤器
                .and()
                .addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        //有动态权限配置时添加动态权限校验过滤器
        if(dynamicSecurityService!=null){
            registry.and().addFilterBefore(dynamicSecurityFilter(), FilterSecurityInterceptor.class);
        }
```
- 不需要保护的资源路径允许访问 
- 允许跨域请求的OPTIONS请求
-  所有请求都要验证
-  关闭跨站请求防护及不使用session
- 在`UsernamePasswordAuthenticationFilter`过滤器前执行`jwtAuthenticationTokenFilter`过滤器

参考资料
- [Spring Boot集成Spring Security](https://www.jianshu.com/p/08cc28921fd0)
- [Spring securing ](https://spring.io/guides/gs/securing-web/)
- [Spring securing 执行顺序](https://www.jianshu.com/p/ac42f38baf6e)

#### JWT
> 本文主要讲解mall通过整合SpringSecurity和JWT实现后台用户的登录和授权功能，同时改造Swagger-UI的配置使其可以自动记住登录令牌进行发送

- JWT token的格式：header.payload.signature
- header中用于存放签名的生成算法 {"alg": "HS512"}
- payload中用于存放用户名、token的生成时间和过期时间   {"sub":"admin","created":1489079981393,"exp":1489684781}
- signature为以header和payload生成的签名，一旦header和payload被篡改，验证将失败 
  String signature = HMACSHA512(base64UrlEncode(header) + "." +base64UrlEncode(payload),secret
  
**JWT实现认证和授权的原理**
- 用户调用登录接口，登录成功后获取到JWT的token；
- 之后用户每次调用接口都在http的`header`中添加一个叫`Authorization`的头，值为JWT的`token`；
- 后台程序通过对`Authorization`头中信息的解码及数字签名校验来获取其中的用户信息，从而实现认证和授权。 

参考资料 [mall整合SpringSecurity和JWT实现认证和授权（一）](http://www.macrozheng.com/#/architect/mall_arch_04)

### AOP
AOP:让`关注点代码`与`业务代码`分离,指对很多功能都有的`重复的代码`抽取，再在运行的时候往业务方法上动态植入“切面类代码”
我们最终的效果是 业务代码只写业务代码  `异常` `日志` `统计`等需要重复写代码的都可以交给`AOP`去统一处理，`filter` `Interceptor`等技术都是切面的应用   
**知识点**
- `@Aspect`  指定一个类为切面类
- `@Pointcut("execution(* cn.itcast.e_aop_anno..(..))")` 指定切入点表达式  即在**哪些类中引入切面**  定义目标方法
- `@Before("pointCut_()") ` 前置通知：目标方法之前执行
- `@After("pointCut_()") ` 后置通知：在目标方法执行后执行该方法 **始终执行**
- `@AfterReturning("pointCut_()")` 返回后通知： 执行方法结束前执
- `@AfterThrowing("pointCut_()")`  异常通知：出现异常时候执行
- `@Around("pointCut_()")`  环绕通知： 环绕目标方法执行  目标放在在该方法中调用执行 **常用的方法**

```
    // 前置通知 : 在执行目标方法之前执行
    @Before("pointCut_()")
    public void begin(){
        System.out.println("开始事务/异常");
    }

    // 后置/最终通知：在执行目标方法之后执行  【无论是否出现异常最终都会执行】
    @After("pointCut_()")
    public void after(){
        System.out.println("提交事务/关闭");
    }

    // 返回后通知： 在调用目标方法结束后执行 【出现异常不执行】
    @AfterReturning("pointCut_()")
    public void afterReturning() {
        System.out.println("afterReturning()");
    }

    // 异常通知： 当目标方法执行异常时候执行此关注点代码
    @AfterThrowing("pointCut_()")
    public void afterThrowing(){
        System.out.println("afterThrowing()");
    }

    // 环绕通知：环绕目标方式执行
    @Around("pointCut_()")
    public void around(ProceedingJoinPoint pjp) throws Throwable{
        System.out.println("环绕前....");
        pjp.proceed();  // 执行目标方法
        System.out.println("环绕后....");
    }
```
正常功能编写代码：每次写`Before`、`After`等，都要重写一次切入点表达式，这样就不优雅了
```
    @Before("execution(* aa.*.*(..))")
    public void begin() {
        System.out.println("开始事务");
    }

    @After("execution(* aa.*.*(..))")
    public void close() {
        System.out.println("关闭事务");
    }
```
我们可以利用`@Pointcut`这个注解来指定切入点表达式，在用到的地方中引用。
```
@Component
@Aspect//指定为切面类
@Order(1)  // 指定执行顺序 同一个方法下应用多个切面时，通过该顺序来执行
public class AOP {
    // 指定切入点表达式，拦截哪个类的哪些方法
    @Pointcut("execution(* aa.*.*(..))")
    public void pt() {

    }
    @Before("pt()")
    public void begin() {
        System.out.println("开始事务");
    }

    @After("pt()")
    public void close() {
        System.out.println("关闭事务");
    }
}
```

参考资料：
- [Spring AOP就是这么简单啦](https://juejin.im/post/5b06bf2df265da0de2574ee1)
- [Spring【AOP模块】就这么简单](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483954&idx=1&sn=b34e385ed716edf6f58998ec329f9867&chksm=ebd74333dca0ca257a77c02ab458300ef982adff3cf37eb6d8d2f985f11df5cc07ef17f659d4#rd)
- [SpringBoot应用中使用AOP记录接口访问日志](http://www.macrozheng.com/#/technology/aop_log)

### 跨域问题
CORS全称Cross-Origin Resource Sharing，意为跨域资源共享 域的定义统一服务(ip+port) 当**一个资源**去访问**另一个域名或者同域名不同端口**的时候，就出现跨域需求   
跨域的请求分两步：
- 先发起一次`OPTIONS`请求进行预检 看是否访问权限
- 发起真实的跨域请求

options预检请求头信息
```
Access-Control-Request-Headers: content-type   
Access-Control-Request-Method: POST  // 表明请求方法
Origin: http://localhost:8090   // 表明请求origin 
Referer: http://localhost:8090/
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36
```
响应头信息
```
Access-Control-Allow-Credentials: true    //这四个字段才是重点
Access-Control-Allow-Headers: content-type
Access-Control-Allow-Methods: POST
Access-Control-Allow-Origin: http://localhost:8090

Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Content-Length: 0
Date: Sat, 27 Jul 2019 13:40:32 GMT
Expires: 0
Pragma: no-cache
Vary: Origin, Access-Control-Request-Method, Access-Control-Request-Headers
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```
spring boot的解决思路：
第一步:定义一个全局覆盖默认的CorsFilter类
```
@Configuration
public class GlobalCorsConfig {

    /**
     * 允许跨域调用的过滤器
     */
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        //允许所有域名进行跨域调用
        config.addAllowedOrigin("*");
        //允许跨越发送cookie
        config.setAllowCredentials(true);
        //放行全部原始头信息
        config.addAllowedHeader("*");
        //允许所有请求方法跨域调用
        config.addAllowedMethod("*");
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
```
设置SpringSecurity允许OPTIONS请求访问 否则预检请求不能正常通过
```
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry registry = httpSecurity
                .authorizeRequests();
        //不需要保护的资源路径允许访问
        for (String url : ignoreUrlsConfig().getUrls()) {
            registry.antMatchers(url).permitAll();
        }
        //允许跨域请求的OPTIONS请求
        registry.antMatchers(HttpMethod.OPTIONS)    
                .permitAll();
```

### mall-admin-web
> mall前端项目
#### Promise异步编程
- 登录远程调用方法封装到Promise中
- 组件(控制层)在调用异步方法
```js
  actions: {
    // 登录远程调用方法封装到Promise中
    Login({ commit }, userInfo) {
      const username = userInfo.username.trim()
      return new Promise((resolve, reject) => {
        login(username, userInfo.password).then(response => {
          const data = response.data
          const tokenStr = data.tokenHead+data.token
          setToken(tokenStr)
          commit('SET_TOKEN', tokenStr)
          resolve()
        }).catch(error => {
          reject(error)
        })
      })
    },
```
组件中调用异步`login` `then`处理异步的返回结果 `catch`处理异步异常的结果
```js
          this.$store.dispatch('Login', this.loginForm).then(() => {
              this.loading = false;
              setCookie("username",this.loginForm.username,15);
              setCookie("password",this.loginForm.password,15);
              this.$router.push({path: '/'})
            }).catch(() => {
              this.loading = false
         })
```
### axios 拦截器
> 对所有axios的请求都带上token字段，要实现这个功能可在axios进行拦截器处理,打上`token`信息 我们也需要对返回的报错信息进行 统一处理 
- 请求拦截器对所有的api请求都打上token信息
- 响应拦截器是对返回状态码进行统一处理，`非200`都抛出错误

```js
// request拦截器
service.interceptors.request.use(config => {
  if (store.getters.token) {
    config.headers['Authorization'] = getToken() // 让每个请求携带自定义token 请根据实际情况自行修改
  }
  return config
}, error => {
  // Do something with request error
  console.log(error) // for debug
  Promise.reject(error)
})

// respone拦截器
service.interceptors.response.use(
  response => {
  /**
  * code为非200是抛错 可结合自己业务进行修改
  */
    const res = response.data
    if (res.code !== 200) {
      Message({                   //这里使用element ui的message组件提示报错信息
        message: res.message,
        type: 'error',
        duration: 3 * 1000
      })

      // 401:未登录;   //这里使用element ui的MessageBox组件提示报错信息
      if (res.code === 401) {
        MessageBox.confirm('你已被登出，可以取消继续留在该页面，或者重新登录', '确定登出', {
          confirmButtonText: '重新登录',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          store.dispatch('FedLogOut').then(() => {
            location.reload()// 为了重新实例化vue-router对象 避免bug
          })
        })
      }
      return Promise.reject('error')
    } else {
      return response.data
    }
  },
  error => {
    console.log('err' + error)// for debug
    Message({
      message: error.message,
      type: 'error',
      duration: 3 * 1000
    })
    return Promise.reject(error)
  }
)
```



### Vue:router的beforeEach与afterEach钩子函数
> `beforeEach`钩子函数会在路由组件匹配之前运行，`afterEach`在路由组件匹配之后处理，类似filter，每次进入vue-router处理流程都会调用这个方法，
往往用来做一些特殊处理
1.通过`token`判断用户是否已登录，如果未登陆，重定向到`login`页面  见`permission.js`
```js
router.beforeEach((to, from, next) => {
  console.log("beforeEach start")
  NProgress.start()
  if (getToken()) {       // 是否存在token
    console.log("beforeEach token")
    if (to.path === '/login') {
      console.log("beforeEach  login start")
      next({ path: '/' })     //重定向到首页
    }
    else {      //不存在token的情况     
    console.log("no token")
    if (whiteList.indexOf(to.path) !== -1) {
      console.log("login in whitelist")
      next()
    } else {
      console.log("redirect login")
      next('/login')
      NProgress.done()
    }
```
2. 根据用户信息生成动态路由



### element-ui组件
> 该项目中用到的ui组件
#### Message MessageBox 用来做消息的提示信息



参考资料：
* [前后端分离项目，如何解决跨域问题](http://www.macrozheng.com/#/technology/springboot_cors?id=%e4%bb%80%e4%b9%88%e6%98%af%e8%b7%a8%e5%9f%9f%e9%97%ae%e9%a2%98)
* [springboot系列文章之实现跨域请求(CORS)](https://juejin.im/post/5b99dcca6fb9a05d3154f8b7)







