### java内存模型演进

#### Java memory model from a logic perspective

![jvm虚拟机模型](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-1.png)

#### Here is a diagram illustrating the call stack and local variables stored on the thread stacks, and objects stored on the heap

![jvm虚拟机模型](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-1.png)


#### Here is a simplified diagram of modern **computer hardware architecture**

![jvm虚拟机模型](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-4.png)

#### Bridging The Gap Between The Java Memory Model And The Hardware Memory Architecture  

As already mentioned, the Java memory model and the hardware memory architecture are different. The hardware memory architecture does 
not distinguish between thread stacks and heap. On the hardware, both the thread stack and the heap are located in main memory. Parts 
of the thread stacks and heap may sometimes be present in CPU caches and in internal CPU registers

![jvm虚拟机模型](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-5.png)

#### Visibility of Shared Objects
The following diagram illustrates the sketched situation. One thread running on the left CPU copies the shared object into its CPU cache,
and changes its count variable to 2. This change is not visible to other threads running on the right CPU, because the update to count    
has not been flushed back to main memory yet.  

![jvm虚拟机模型](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-6.png)

参考链接：  
http://tutorials.jenkov.com/java-concurrency/java-memory-model.html




