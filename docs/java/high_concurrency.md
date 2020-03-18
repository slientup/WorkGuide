- [多线程](#多线程)  
  - [线程的生命周期和状态](#线程的生命周期和状态)
  - [线程状态转换图V2](#线程状态转换图V2)
  - [wait和notify理解](#wait和notify理解)
  

### 多线程
#### 线程的生命周期和状态

Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态
![Java 线程的状态 ](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png)

![Java 线程状态变迁 ](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java+%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%8F%98%E8%BF%81.png)  

由上图可以看出线程状态迁移图：
1. 线程创建之后它将处于`new(新建)`状态，调用`start()`方法后开始运行，线程在这时候处于Ready(就绪)状态，就绪状态的线程获得了cpu时间片(`timeslice`)后就处于`RUNNING(运行)`状态.      
2. 当线程执行wait()方法后，线程就进入**waiting等待阻塞**状态，进入等待阻塞状态的线程需要`其他线程`的通知才能返回到`Ready(就绪)`状态,而time_waitting
(超时等待)状态相当于在等待状态的基础上增加了超时限制，比如通过`sleep(long millis)`方法或`wait(long millis)`方法可以将java线程置于`Time waiting`状态，当超时时间到达后java线程将会返回`Ready(就绪)`状态,当线程调用同步方法时，在没有获得锁的情况下，线程将进入**BLOCKED**阻塞状态。 
这张图更详细简单的描述了这个状态转换关系：

#### 线程状态转换图V2
![Java 线程状态简要图 ](https://github.com/slientup/WorkGuide/blob/master/thread_status.png)

**三种大的状态**  
1. 就绪状态:`READY` 2. 运行状态:`RUNNING` 3. 阻塞状态:`wait` `time_waiting` `blocked`
这三种阻塞状态的区别： 
- time_waitting状态在超时时间到达后**直接**进入`Ready就绪状态`    
- `wait`阻塞状态的线程**不能直接**进入`Ready就绪状态` ,`wait阻塞`状态必须要有`其他线程notifiy`才能唤醒该线程，该线程唤醒后会继续去争取同步锁，若未获取到锁就会进入到**BLOCKED**，只有获取到锁才能进入就绪状态。  
- 若未获取到同步锁synchronized 当前线程就进入**BLOCKED**状态，获得锁后就可以进入就绪状态.  
- 总结：`wait_waiting` `blcoked` 可直接进入`就绪状态`，而`wait状态`的线程不可直接进入。

#### wait和notify理解  
void notify()  Wakes up a `single thread` that is waiting on this `object’s` monitor.   
唤醒在此对象监视器上等待的单个线程  
void notifyAll() Wakes up `all threads` that are waiting on this object’s monitor.  
唤醒在此对象监视器上等待的所有线程  
void wait() Causes the current thread to wait until another thread invokes the notify() method or the notifyAll( ) method for this object.  
wait方法会让当前的线程等待，直到其他线程调用这对象的notify( ) 方法或 notifyAll( ) 方法 
重点解释：
1. wait() notify() notifyAll()都不属于thread类，而是属于Object基础类，也就是每个对象都有这三种方法，当然每个对象都有锁。  
2. 这三种方法必须要加锁，否则就会出现IllegalMonitorStateException异常;  
3. wait() notify() notifyAll() 必须放到synchronized(obj)对象里，否则会出现IllegalMonitorStateException异常;  
4. 假设有三个线程执行了obj.wait(),使用obj.notifyall会唤醒这三个线程，他们会去竞争锁，只有获得锁的线程才会执行obj.wait()的下一条语句，
其余的依然等待;  
5.当调用`obj.notify/notifyAll`后，`调用线程`依然持有`obj`锁，因此，thread1、thread2和thread3虽然被唤醒，但是只有当这个线程释放锁后，他们
才有机会获得该锁obj对象锁;  






