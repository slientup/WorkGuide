- [监听器原理](#监听器原理)
- [源码分析](#源码分析)

### 监听器原理

1. 客户端有一个主线程    
2. 主线程创建一个eventThread()和sendThread()  
3. 一个负责监听，一个负责网络连接  
4. 通过connect线程将注册的监听事件发送给zookeeper  
5. 在zookeeper的注册监听器列表中将注册的监听事件添加到列表中  
6. zookeeper监听有数据或路径发生变化，就将这个消息发送给监听线程  
7. listenner线程调用Process()方法  
常见监听：监听节点数据的变化  
         监听子节点增减的变化
     
### 源码分析
- [zookeeper源码分析](http://www.cnblogs.com/leesf456/p/6518040.html)
