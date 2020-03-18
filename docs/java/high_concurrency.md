- [多线程](#多线程)  
  - [线程的生命周期和状态](#线程的生命周期和状态)
  

### 多线程
#### 线程的生命周期和状态

Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态
![Java 线程的状态 ](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png)

![Java 线程状态变迁 ](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java+%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%8F%98%E8%BF%81.png)
由上图可以看出线程状态迁移图：
1. 线程创建之后它将处于`new(新建)`状态，调用`start()`方法后开始运行，线程在这时候处于Ready(就绪)状态，就绪状态的线程获得了cpu时间片(`timeslice`)后就处于`RUNNING(运行)`状态.      
2. 当线程执行wait()方法后，线程就进入`waiting等待阻塞`状态，进入等待阻塞状态的线程需要`其他线程`的通知才能返回到`Ready(就绪)`状态,而time_waitting
(超时等待)状态相当于在等待状态的基础上增加了超时限制，比如通过`sleep(long millis)`方法或`wait(long millis)`方法可以将java线程置于Time waiting状态
，当超时时间到达后java线程将会返回`Ready(就绪)`状态,当线程调用同步方法时，在没有获得锁的情况下，线程将进入`BLOCKED`阻塞状态。  

