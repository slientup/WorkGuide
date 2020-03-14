Rabbitmq 

- [RabbitMq](# RabbitMq)
- [Kafka](# Kafka)
- [两者对比](# 两者对比)
    - [架构模型](# 架构模型)
    - [消息顺序](# 消息顺序)
    - [消息确认机制](# 消息确认机制)
    - [吞吐量](# 吞吐量)
    

### RabbitMq
待写.....

### Kafka
待写.....

### 两者对比

#### 架构模型 

`RabbitMq`: `Push`消息模型，采用`监听器`的方式，当队列中存在消息，通知订阅者来获取，以broker为中心
`Kafka`: `Pull`消息模型，consumer一直循环拉取队列里面的数据，即使没有数据也会去拉，以consumer为中心，






