- 1. 通过rancher的app直接部署
- 2. 通过ingress发布出去

在第二步通过ingress发布出去的时候，出现了一些问题

1. 将15672端口和5672端口同时通过ingress暴露给外部环境，但只有15672端口生效了，5672端口并不是nginx协议，因此5672端口不能通过ingress的方式发不出去吗？
2. 经过这个想法之后，我通过内部域名的方式访问到rabbitmq服务， 域名规则https://jimmysong.io/kubernetes-handbook/concepts/statefulset.html  

域名规则：
StatefulSet中每个Pod的DNS格式为statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local，其中

- serviceName为Headless Service的名字
- 0..N-1为Pod所在的序号，从0开始到N-1
- statefulSetName为StatefulSet的名字
- namespace为服务所在的namespace，Headless Servic和StatefulSet必须在相同的namespace
- .cluster.local为Cluster Domain

实际我只配置了statefulSetName-{0..N-1}.serviceName.namespace 就生效了 alertcenter-rabbitmq-0.alertcenter-rabbitmq.rabbitmq
