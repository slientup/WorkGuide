#### java中Log系统比较
参考链接:    
- [Java日志框架SLF4J和log4j以及logback的联系和区别](https://www.cnblogs.com/hanszhao/p/9754419.html) 
- [logback](http://www.logback.cn/)
### 简介
#### SLF4J(Simple logging Facade for Java)
简单的日志门面，`只提供了统一的日志使用接口`，没有具体的实现，具体的`实现日志系统`有`log4j` `logback`等      
SLF4j由于只是一个接口，要实现日志功能，需要调用对应的实现日志系统，需要添加对应jar包。  
- SLF4J和logback结合使用时需要提供的jar:slf4j-api.jar,logback-classic.jar,logback-core.jar  
- SLF4J和log4j结合使用时需要提供的jar:slf4j-api.jar,slf4j-log412.jar,log4j.jar
  
**注意，以上slf4j和各日志实现包结合使用时最好只使用一种结合，不然的话会提示重复绑定日志，并且会导致日志无法输出.**   

slf4j-api.jar:对外提供统一的日志调用接口，该接口具体提供的调用方式和方法举例说明：   
```
public class Test {

　　private static final Logger logger = LoggerFactory.getLogger(Tester.class);  //通过LoggerFactory获取Logger实例
  
　　public static void main(String[] args) {
       //接口里的统一的调用方法，各具体的日志系统都有实现这些方法
       logger.debug("testlog: {}", "test");  
       logger.warn("testlog: {}", "test");
　　}
}
```
**推荐使用格式化的方式进行替换，而不是+拼接字符串**
#### Log4j
是对slf4j的日志实现系统，可以控制日志信息输送的目的地是控制台、文件、GUI组件。
#### Logback
是对log4j的改进，作者一样，性能更优越，是完全实现slf4j的接口。
#### 总结
1. slf4j是java的一个日志门面，实现了日志框架一些通用的api，log4j和logback是具体的日志框架  
2. 他们可以单独的使用，也可以绑定slf4j一起使用
#### 推荐方案(也是阿里巴巴推荐的方案)
`slf4j`+`logback` 不用log4j 为什么要使用SLF4J?   
- slf4j是一个日志接口，自己没有具体实现日志系统，只提供了一组标准的调用api,这样将调用和具体的日志实现分离，使用slf4j后有利于根据自己实际的需求
更换具体的日志系统，比如，之前使用的具体的日志系统为log4j,想更换为logback时，只需要删除log4j相关的jar,然后加入logback相关的jar和日志配置文件即可，
而不需要改动具体的日志输出方法，试想如果没有采用这种方式，当你的系统中日志输出有成千上万条时，你要更换日志系统将是多么庞大的一项工程。如果你开发的
是一个面向公众使用的组件或公共服务模块，那么一定要使用slf4的这种形式，这有利于别人在调用你的模块时保持和他系统中使用统一的日志输出。
- slf4j日志输出时可以使用{}占位符，如，logger.info("testlog: {}", "test")，而如果只使用log4j做日志输出时，只能以logger.info("testlog:"+"test")
这种形式，前者要比后者在性能上更好，后者采用+连接字符串时就是new 一个String 字符串，在性能上就不如前者。





