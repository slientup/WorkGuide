## spring boot 常用功能实现
- [定时任务](#定时任务)
- [异步](#异步)
- [long polling](#long polling)
- [参考链接](#参考链接)




### 定时任务
#### 原理  
最终委托调用的是并发类的`Executors.newSingleThreadScheduledExecutor()`来创建一个**新的线程**来执行这个定时任务  
#### 使用方法   
使用非常简单，只需要这两步  
- `@EnableScheduling` 只需要在SpringBoot的*Application.java文件上加入`@EnableScheduling`注解即可使用Schedule  
- `@Scheduled` 使用该注解作用于方法，该方法就会执行定时任务了 @Scheduled(fixedDelay = 2000)

        @EnableScheduling
        @SpringBootApplication
        public class ScheduleApplication {
            public static void main(String[] args) {
                SpringApplication.run(ScheduleApplication.class, args);
            }
        }
        @Scheduled(fixedDelay = 2000)
        public void fixedRateTask() {
            taskService.sayHello("fixedDelay");
        }


### 异步
#### 原理 
最终调用`Executors SimpleAsyncTaskExecutor` 启动一个新的线程,执行该方法,异步本质就是起一个新的线程；
#### 使用方法 
- `@EnableAsync`：通过在配置类或者`Main`类上加`@EnableAsync`开启对异步方法的支持。
- `@Async` 可以作用在类上或者方法上，作用在类上代表这个类的所有方法都是异步方法
- 通过该类`AsyncResult`返回异步的结果  实际是Future类;
- 主类通过 `Future.get()`查看返回的结果

        @Component
        public class TestAsyncBean {
            **@Async**
            public Future<String> sayHello1() throws InterruptedException {
                int thinking = 2;
                Thread.sleep(thinking * 1000);//网络连接中 。。。消息发送中。。。
                System.out.println("我爱你啊!");
                return new AsyncResult<String>("发送消息用了"+thinking+"秒");
            }
        }
        @RunWith(SpringJUnit4ClassRunner.class)
        @ContextConfiguration({"classpath:/applicationContext.xml"})
        public class TestAsync {
            @Autowired
            private TestAsyncBean testAsyncBean;
            @Test
            public void test_sayHello1() throws InterruptedException, ExecutionException {
                Future<String> future = null;     // 定义future类
                System.out.println("你不爱我了么?");
                future = testAsyncBean.sayHello1();
                System.out.println("你竟无话可说, 我们分手吧。。。");
                Thread.sleep(3 * 1000);// 不让主进程过早结束
                System.out.println(future.get());   //获取返回结果
            }
        }
### long polling 
spring mvc的long polling  
- `polling`：如果我想在两分钟内看到快递的变化，那么，轮询会每隔两分钟去像服务器发起一次快递变更的查询请求，如果快递其实是一个小时变更一次，那么-  polling的方式在获取一次真实有效信息时需要发起30次  
- `long polling`：**首先发起查询请求，服务端没有更新的话就不回复，直到一个小时变更时才将结果返回给客户，然后客户发起下次查询请求**。长轮询保证了每次发起的查询请求都是有效的，**极大的减少了与服务端的交互**，基于web异步处理技术，大大的提升了服务性能.  
#### 应用场景
**发布订阅**的例子：配置中心服务，当配置中心的配置变更好，相关的客户端程序需要及时更新最新的配置。`disconf`就是基于`zookeeper的发布订阅`来做的，apollo就是采用的`DeferredResult`的`long polling`来做的，客户端发起长轮询，配置中心监听器监听到配置变更后，将结果响应给客户端.



### 参考链接
-[使用@Async进行异步调用详解](https://juejin.im/post/5b27b8366fb9a00e46675879)  
-[SpringBoot中的Schedule](https://juejin.im/post/5d1c07875188255100080b12)
-[springMvc DeferredResult的long polling应用](http://www.kailing.pub/article/index/arcid/163.html)



