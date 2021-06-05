
  * [Promethouse+alertmanager告警测试](#Promethouse+alertmanager告警测试)
  * [客户端接入案例](#客户端接入案例)
  * [promethous api查询数据](promethousapi查询数据)

### Promethouse+alertmanager告警测试
#### 文档
- [promethouse 文档](https://yunlzheng.gitbook.io/prometheus-book/part-iii-prometheus-shi-zhan/readmd/service-discovery-with-kubernetes)
- [Prometheus 监控报警系统 AlertManager 之邮件告警](https://cloud.tencent.com/developer/article/1486483)

#### 启动Promethouse
`docker run -p 9090:9090 -v $home/config.txt:/etc/prometheus/prometheus.yml prom/prometheus --config.file=/etc/prometheus/prometheus.yml --web.enable-lifecycle`
#### 启动alertmanager
`docker run -d --name alertmanager -p 9093:9093 -v $home/alertmanager.yml:/etc/alertmanager/alertmanager.yml prom/alertmanager:latest`


#### promethouse 告警相关的配置
```
alerting:
  alertmanagers:
  - follow_redirects: true
    scheme: http
    timeout: 10s
    api_version: v2
    static_configs:
    - targets:
      - xxxx:9093  // 指定alertmanger的地址  
rule_files:
- /etc/prometheus/rules/*.rules  //告警规则配置位置
```

rules内容 /etc/prometheus/rules/test.rules

```yaml
groups:
- name: example
  rules:
  # alert for instance down 
  - alert: InstanceDown
    expr: up == 0
    labels:
      severity: P3   //自定义的参数
      namespace: test  // 
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      value: "{{ $value }}"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
```

#### alertmanger 告警相关的配置

只在文字说明处做了修改
```yaml
global:
  resolve_timeout: 5m
  http_config: {}
  smtp_hello: localhost
  smtp_require_tls: true
  pagerduty_url: https://events.pagerduty.com/v2/enqueue
  opsgenie_api_url: https://api.opsgenie.com/
  wechat_api_url: https://qyapi.weixin.qq.com/cgi-bin/
  victorops_api_url: https://alert.victorops.com/integrations/generic/20131114/alert/
route:
  receiver: web.hook
  group_by:
  - alertname
  group_wait: 10s    
  group_interval: 10s
  repeat_interval: 10s   //重复告警静默时间改成10s 随时都发送
inhibit_rules:
- source_match:
    severity: critical
  target_match:
    severity: warning
  equal:
  - alertname
  - dev
  - instance
receivers:
- name: web.hook
  webhook_configs:
  - send_resolved: true
    http_config: {}
    url: http://xxxx:8080/api/customizedAlert/alert_manager/   //只做了这一个改动
    max_alerts: 0
templates: []
```

### 客户端接入案例

启动pushgateway `docker run -d -p 9091:9091 prom/pushgateway`

- python 推送自定义标签数据
    ```python
  from prometheus_client import CollectorRegistry, Gauge, push_to_gateway


  def push_to_gateway_label(value):
      """
      以下生成的metric为：
      http_request_status{endpoint="/hello",instance="192.168.1.2:9100",job="test"，status_code="200"} 1
      """
      registry = CollectorRegistry()
      # 自定义标签
      labels = ['instance', 'endpoint', 'status_code']
      g = Gauge('http_request_status', 'status if normal', labels, registry=registry)
      # 标签赋值
      g.labels('192.168.1.2:9100', '/hello', 200).set(value)
      # 127.0.0.1:9091 pushgateway地址
      push_to_gateway('127.0.0.1:9091', job='test', registry=registry)


  if __name__ == '__main__':
      push_to_gateway_label(value=1)
    ```
- java 推送自定义标签数据(与python同一组数据)
  ```java
  import io.prometheus.client.CollectorRegistry;
  import io.prometheus.client.Gauge;
  import io.prometheus.client.exporter.PushGateway;

  import java.io.IOException;

  public class PushToGateway {
      public static void main(String[] args) throws IOException {
          CollectorRegistry registry = new CollectorRegistry();
          Gauge gauge = Gauge.build()
                  .name("http_request_status").help("status if normal").
                          labelNames("instance", "endpoint", "status_code").register(registry);
          gauge.labels("192.168.1.2:9100", "/hello", "200").set(1);
          PushGateway pg = new PushGateway("127.0.0.1:9091");
          pg.pushAdd(registry, "test456");
      }
  }
  ```

#### 参考文档
- [不同客户端使用案例参考连接](https://prometheus.io/docs/instrumenting/pushing/)

### promethous api 查询数据

查询语句：
`1 - sum ( node_memory_MemAvailable{instance=~"10.24.51.155:9100"} ) / sum (node_memory_MemTotal{instance=~"10.24.51.155:9100"})
`

api url：
`http://localhost:9090/api/v1/query?query=`

最终调用api
`api url`+`查询语句`

`http://localhost:9090/api/v1/query?query=1 - sum ( node_memory_MemAvailable{instance=~"10.24.51.155:9100"} ) / sum (node_memory_MemTotal{instance=~"10.24.51.155:9100"})`


参考链接：
`https://prometheus.io/docs/prometheus/latest/querying/api/`


