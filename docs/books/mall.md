# 开源项目mall 源码阅读笔记
* [mall-common](#mall-common)
* [mall-admin](#mall-admin)

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
参考资料：[spring自定义vaildation](https://snailclimb.gitee.io/springboot-guide/#/./docs/advanced/spring-bean-validation)





