## grafana webhook
#### 背景
目前消息队列等中间件产品采集指标时，主要通过`promethouse exporter`的方式采集，`grafana`的方式展示，但原生解决方案采用的`alermanger`告警，但`alertmanger`并不提供web配置，这对开发人员定制化告警就很不方便

#### 解决思路
在grafana增加`webhook`通知模式，webhook指向告警中台。

#### 适应场景
主要用于`kafka`、`rabbitmq`等消息队列以及其他的一些特殊场景。

#### 通知消息体json
```
{
  "instance": "xxx", // 告警的实例
  "subject": "time out", //告警内容
  "Receiver"：["xxx"], // 告警接收人list，
  "notificationWay":["email","wechat"] //通知方式list
}
```
#### 案例
**1**

需求：当rabbitmq消息队列积压最近5分钟平均值超过`1000`邮件告警通知到xxx,配置如下

![](https://files.mdnice.com/user/4251/1ce113a0-61a2-48ff-a855-d0ed71f724db.png)

#### 注意事项

- **不配置for**  立即发送告警
- **配置for** 观察一段时间后，如果告警持续则发送，恢复则不发送。

以下是官方文档关于该字段解释：

`For - Specify how long the query needs to violate the configured thresholds before the alert notification triggers.`

`If an alert rule has a configured For and the query violates the configured threshold, then it will first go from OK to Pending. Going from OK to Pending Grafana will not send any notifications. Once the alert rule has been firing for more than For duration, it will change to Alerting and send alert notifications`
