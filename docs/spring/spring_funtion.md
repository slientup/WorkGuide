## spring 常用功能实现
- [定时任务](#定时任务)
- [异步](#异步)



### 定时任务
#### 原理  
最终委托调用的是并发类的`Executors.newSingleThreadScheduledExecutor()`来创建一个**新的线程**来执行这个定时任务  
#### 使用方法   
待写...



### 异步
#### 原理 
最终调用`Executors SimpleAsyncTaskExecutor` 启动一个新的线程,执行该方法,异步本质就是起一个新的线程；
#### 使用方法 
- `public` 方法上直接引入 `@Async` 
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



