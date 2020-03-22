- [AQS使用场景和含义](#AQS使用场景和含义)
- [CountDownLatch(倒计时器)](#CountDownLatch(倒计时器))

参考链接 
   - 深入分析AQS实现原理  https://segmentfault.com/a/1190000017372067
   - AQS 简单介绍和CountDownLatch （倒计时器）https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/AQS.md#3semaphore%E4%BF%A1%E5%8F%B7%E9%87%8F%E5%85%81%E8%AE%B%E5%A4%9A%E4%B8%AA%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%97%B6%E8%AE%BF%E9%97%AE
  


### AQS使用场景和含义
在工作中，我们为保证线程安全，常常会使用同步synchronized，但也还有一种实现方式Lock。
lock的简单demo：
```
public class Demo {
    private static int count=0;
    static Lock lock=new ReentrantLock();  //定义lock对象
    public static void inc(){
        lock.lock();    //获取lock
        try {
            Thread.sleep(1);
            count++;
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally{
            lock.unlock();  //释放锁
        }
    }
```
**问题**：当多个线程通过lock竞争锁的时候，那些没有获得锁的线程是如何**实现等待**并**如何唤醒**的啦？

AQS：AbstractQueuedSynchronizer  AQS依赖内部的FIFI双向队列，如果当前线程竞争锁失败，那么AQS会把当前线程以及等待状态信息构造成一个Node加入
到同步队列中，同时再阻塞该线程，当获取锁的线程释放锁以后，会从队列中唤醒一个阻塞的节点(线程)
![原理图](https://camo.githubusercontent.com/55090fdc22963d41a3e56d5dadb17dfa7ad6379f/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f4a6176612532302545372541382538422545352542412538462545352539312539382545352542462538352545352541342538372545462542432539412545352542392542362545352538462539312545372539462541352545382541462538362545372542332542422545372542422539462545362538302542422545372542422539332f434c482e706e67)

从使用层面来说，AQS的功能分为两种：独占和共享  
- 独占：只允许一个线程能获得锁   比如ReentrantLock  
- 共享: 可以允许多个线程同时获得锁 `并发访问共享资源`  比如`ReentrantReadWriteLock`  `Semaphore/CountDownLatch` `CyclicBarrier`


锁可以分为两种情况:
- **公平锁**：按照线程在队列中的排队顺序，先到者先拿到锁  
- **非公平锁**：当线程要获取锁时，先通过两次 CAS 操作去抢锁，如果没抢到，当前线程再加入到队列中等待唤醒  

### CountDownLatch(倒计时器)
CountDownLatch允许`count`个线程阻塞在一个地方，**直至所有线程的任务都执行完毕**   
**运行原理**：CountDownLatch是共享锁的一种实现,它默认构造 AQS 的 `state` 值为 `count`。当线程使用countDown方法时,其实使用了tryReleaseShared  
方法以CAS的操作来减少state,直至state为0就代表所有的线程都调用了`countDown`方法。当调用`await`方法的时候，如果state不为0，`就代表仍然有线程没有     
调用countDown方法`，那么就把已经调用过countDown的线程都放入阻塞队列Park,并自旋CAS判断state == 0，直至最后一个线程调用了countDown，使得state      
== 0，于是阻塞的线程便判断成功，全部往下执行。  

`CountDownLatch`和`CyclicBarrier`非常像，应用场景也很类似，只是后者是`cycle`循环的，`count`计数可重复使用。

**CountDownLatch两种典型运用场景**

- **某一线程在开始运行前等待 n 个线程执行完毕**。将 CountDownLatch 的计数器初始化为 n ：new CountDownLatch(n)，每当一个任务线程执行完毕，就将计数器
减 1 countdownlatch.countDown()，当计数器的值变为 0 时，在CountDownLatch上 await() 的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程
需要等待多个组件加载完毕，之后再继续执行。
- **实现多个线程开始执行任务的最大并行性**。注意是`并行性，不是并发`，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，
然后同时开跑。做法是初始化一个共享的 CountDownLatch 对象，将其计数器初始化为 1 ：new CountDownLatch(1)，多个线程在开始执行任务前首先 
coundownlatch.await()，当主线程调用 countDown() 时，计数器变为 0，多个线程同时被唤醒。
**场景1** 常常是分布式计算后再分开，即任务切分，再组装，可使用**java多线程中join方法**也可以实现  
**场景2** 常常用来模拟高并行，对应用程序做并行测试  

demo
```
/**
 *
 * @author SnailClimb
 * @date 2018年10月1日
 * @Description: CountDownLatch 使用方法示例
 */
public class CountDownLatchExample1 {
  // 请求的数量
  private static final int threadCount = 550;

  public static void main(String[] args) throws InterruptedException {
    // 创建一个具有固定线程数量的线程池对象（如果这里线程池的线程数量给太少的话你会发现执行的很慢）
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
    for (int i = 0; i < threadCount; i++) {
      final int threadnum = i;
      threadPool.execute(() -> {// Lambda 表达式的运用
        try {
          test(threadnum);
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        } finally {
          countDownLatch.countDown();// 表示一个请求已经被完成
        }

      });
    }
    countDownLatch.await();
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);// 模拟请求的耗时操作
    System.out.println("threadnum:" + threadnum);
    Thread.sleep(1000);// 模拟请求的耗时操作
  }
}
```













