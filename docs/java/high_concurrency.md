- [多线程](#多线程)  
  - [线程的生命周期和状态](#线程的生命周期和状态)
  - [线程状态转换图V2](#线程状态转换图V2)
  - [wait和notify理解](#wait和notify理解)
  - [join方法](#join方法)
  - [synchronized关键字](#synchronized关键字)
  - [总结](#总结)
- [参考](#参考)
  

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
1. `wait() notify() notifyAll()`都不属于thread类，而是属于`Object`基础类，也就是**每个对象都有这三种方法**，当然**每个对象都有锁**。  
2. wait() notify() notifyAll() 必须放到`synchronized(obj)`对象里，否则会出现`IllegalMonitorStateException`异常;  
3. 假设有三个线程执行了obj.wait(),使用obj.notifyall会唤醒这三个线程，他们会去竞争锁，只有获得锁的线程才会执行obj.wait()的下一条语句，
其余的依然等待;  
4. 当调用`obj.notify/notifyAll`后，`调用线程`依然持有`obj`锁，因此，thread1、thread2和thread3虽然被唤醒，但是只有当这个线程释放锁后，他们
才有机会获得该锁obj对象锁;  

#### join方法  
使用场景：**当一个线程必须等待另一个线程执行完毕才能执行时**  
join方法含义： `当前线程`等该`加入该线程`后面，等待该线程终止  
**使用案例**：

        for (int i = 0; i < 5; i++) {
            MyThread thread = new MyThread();
            //启动线程
            thread.start();
            try {
                //调用join()方法
                thread.join();   //主线程等待子线程结束后才继续往下执行
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("主线程执行完毕");
**源码分析**：

    阻塞当前线程,直接调用子线程结束才唤醒  锁对象都是this
    public final synchronized void join(long millis) throws InterruptedException {
           .....省略.....
            if (millis == 0) {  //子线程或者就一直阻塞
                while (isAlive()) { //判断子线程是否存活
                    wait(0); //调用wait(0)方法
                }
            } 
            .....省略.....
            
     void threadTerminated(Thread t) {   //在线程结束时，唤醒线程
        synchronized (this) {
            remove(t);                          //从线程组中删除此线程

            if (nthreads == 0) {                //当线程组中线程数为0时
                notifyAll();                    //唤醒所有待定中的线程
            }
            if (daemon && (nthreads == 0) &&
                (nUnstartedThreads == 0) && (ngroups == 0))
            {
                destroy();
            }
        }
    }

#### synchronized关键字
它包括两种用法：`synchronized 方法`和 `synchronized 块`
1. synchronized 方法：通过在方法声明中加入 synchronized关键字来声明 synchronized 方法, 对象锁是this/当前clas的实例
2. synchronized(this){} 能缩下同步的范围，锁对象也是this
3. 使用案例可参见上面的源码

#### 总结
- wait方法的作用：当前线程阻塞，并释放锁；而sleep方法：并不会释放锁       
- 所以wait通常被用于线程间交互/通信，sleep 通常被用于暂停执行，而不用于同步锁的场景    
- synchronized 同步锁内可以没有wait notify通知方法，但一旦有wait notify方法的话，必须置于同步锁内；  
- java底层通过Object的wait和notify/notify和synchronized实现线程之间的等待/通知机制，而Condition与Lock await signal配合也能完成等待通知机制
，后者能做到更多事情，并发容器就使用后者来实现进程之间通信。  

在实际使用中，我们更多使用的是**封装好的容器和线程池**之类的，但底层原理都是这些。
      
### 参考
- [wait和notify的理解与使用](https://blog.csdn.net/coding_1994/article/details/80634792)
- [join方法](https://juejin.im/post/5b3054c66fb9a00e4d53ef75)
- [详解Condition的await和signal等待/通知机制](https://www.jianshu.com/p/28387056eeb4)






