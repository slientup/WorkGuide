
- [RabbitMq](#RabbitMq)
   - [常用命令](#常用命令)
- [Kafka](#Kafka)
- [两者对比](#两者对比)
    - [架构模型](#架构模型)
    - [消息顺序](#消息顺序)
    - [消息确认机制](#消息确认机制)
    - [吞吐量](#吞吐量)
- [参考链接](#参考链接)  

### RabbitMq
#### 常用命令
- service rabbitmq-server start 启动服务
- chkconfig rabbitmq-server on  开机启动
- service rabbitmq-server status 查看服务状态
- rabbitmq-plugins enable rabbitmq_management  开启web管理插件
- rabbitmqctl 是rabbitmq的管理工具命令
#### Erlang节点和Erlang应用程序
rabbitmq依赖于Erlang 在Erlang中节点和应用程序的区别：就好比jvm和java程序的区别 应用程序必须运行在节点之上，只有`同时都在才能运行`  
- rabbitmqctl stop 关闭节点和应用
- rabbitmqctl stop_app 只关闭节点
在实际应用中，主要使用rabbitmqctl stop/start就ok了

### Kafka 
待写.....

### 两者对比

#### 架构模型 

`RabbitMq`: `Push`消息模型，采用`监听器`的方式，当队列中存在消息，通知订阅者来获取，以`broker`为中心  
`Kafka`: `Pull`消息模型，consumer一直循环拉取队列里面的数据，即使没有数据也会去拉，以`consumer`为中心
#### 消息顺序
`RabbitMq`: 同一个队列里的消息，严格按照先进先出  
`Kafka`: 同一个`partition`里面的消息，保证先进先出，可在生产者端指定partition，或者通过key的方式

#### 消息确认机制
`RabbitMq`: 生产者通过confirm机制，消费者通过应答机制来确认
`Kafka`: 消费者无确认机制

#### 吞吐量
`RabbitMq`: 吞吐量很低
`Kafka`: 具有极大的吞吐量，能达到百万/s

#### 使用场景
`RabbitMq`: 推送模式，吞吐量小，完善的确认机制，适合处理实时业务场景；  
`Kafka`: 适合处理海量数据的场景，如监控，日志  
`push模式`和`pull模式`**区别**：push只有在队列里面有数据的时候才会通知监控程序过来取数据，pull模式，一直去队列里问是否有数据，不管是否有数据；

### 参考链接
- [RabbitMQ和Kafka的区别](https://www.cnblogs.com/iiwen/p/10195293.html)










