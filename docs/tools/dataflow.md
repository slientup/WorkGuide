#### 访问数据流
k8s服务对集群外发布的数据流：

`pod---service（clusterIP）----ingress(入口)`

#### 高可用
- pod如何保证高可用？

  创建多个`pod`，通过`service`调度
- service如何保证高可用？创建的时候并没有多个选择？service ip如何映射pod ip?service是如何感知到pod ip并更新上面问题当中的映射关系的?

  这与`service`实现原理有关，实际上`service`并不存在`具体的node`里面，而是存在每一个node里面，因为这个负载均衡条目写在了宿主机内核的`ipvs`。这里面发挥作用的就是`kube-proxy`组件。
  
  当我们创建一个Service的时候，一旦它被提交给 Kubernetes，那么`kube-proxy`就可以通过`Service`的`Informer`感知到这样一个 Service 对象的添加。而作为对这个事件的响应，kube-proxy首先会在宿主机上创建一个虚拟网卡（叫作：`kube-ipvs0`)，并为它分配`Service VIP`作为IP地址.

  `ipvsadm -Ln` 查看ipvs

  ```
  [root@k8s-master service]# ipvsadm -Ln
  IP Virtual Server version 1.2.1 (size=4096)
  Prot LocalAddress:Port Scheduler Flags
    -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
  ……………… 
  TCP  10.106.66.120:80 rr
    -> 10.244.2.115:80              Masq    1      0          0         
    -> 10.244.2.116:80              Masq    1      0          0         
    -> 10.244.4.105:80              Masq    1      0          0         
  ```

- `ingress`是如何保证高可用的？
  ingress以`pod`的方式创建，可以部署在多台机器上，比如rancher就在每台机器都部署的，到目前为止没有出集群，为了让集群外的机器能访问到ingress，ingress通过`hostnetwork`的方式将nginx服务暴露出去，而外部dns只需要将泛域名指向任意一台ingress就能实现流量转发，因为任意一台ingress都是k8s的集群。

  **ingress controller的工作原理**
  - `ingress controller`通过和`kubernetes api`交互，动态的去感知集群中`ingress`规则变化;
  - 读取规则，按照自定义的规则，规则就是写明了哪个域名对应哪个service，生成一段`nginx`配置
  - nginx-ingress-controller的pod里，这个Ingress controller的pod里运行着一个Nginx服务，控制器会把生成的nginx配置写入`/etc/nginx.conf`文件
  - `reload`一下使配置生效。以此达到域名分别配置和动态更新的问题。

  ingress中创建`www.test.com`之后，nginx的配置文件就会创建对应的条目
    ```
    server {
                    server_name www.test.com ;

                    listen 80  ;
                    listen [::]:80  ;
                    listen 443  ssl http2 ;
                    listen [::]:443  ssl http2 ;
                    location / {

                            set $namespace      "grafana";
                            set $ingress_name   "grafana";
                            set $service_name   "grafana";
                            set $service_port   "80";
                            set $location_path  "/";
                proxy_pass http://upstream_balancer;
        }
    ```

#### 外部数据流
外部访问集群内部的服务，数据流大体如下：
1. 通过`外部dns服务器`解析到`k8s`集群任意的`ingress`所在机器里，由于域名的后缀是不变的，所以dns域名解析记录使用泛域名解析(同时解析到多台node的宿主机ip地址上)
2. 当流量到达ingress(集群入口)，后续的网络通信，就是集群内部通信。先通过nginx查找该域名对应的service，然后将目的地址转换成该`service_ip`；
3. 当流量转换到`service_ip`之后，会将目的地址替换成对应的pod地址，这一步是在宿主机的内核的`ipvs`表中完成的。

#### 思考

1. 由于k8s设计时，所有用户的操作都是跟API接口对接的，也就是所有的数据信息都在etcd中存储，而外部系统想要知道pod service ingress等对应关系的变化信息，都只需要与k8s api对接获取对应的数据就可以。
   比如`coredns`插件，通过监视`Kubernetes API`并根据`Kubernetes DNS` 规范提供DNS记录填充到本地`corefile`文件里面。

2. k8s里面需要涉及到nat步骤，一般思路都放在`iptable`、`ipvs`中.





