#### 文档
- [promethouse 文档](https://yunlzheng.gitbook.io/prometheus-book/part-iii-prometheus-shi-zhan/readmd/service-discovery-with-kubernetes)
- [Prometheus 监控报警系统 AlertManager 之邮件告警](https://cloud.tencent.com/developer/article/1486483)


### Promethouse+alertmanager 告警测试

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

```
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
```
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



