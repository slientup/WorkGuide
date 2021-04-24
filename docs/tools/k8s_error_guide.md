#### k8s 排障思路
- 查看异常资源对应的状态；
- 通过describe查看更详细的信息和event信息
- 通过pod日志或者容器日志查看具体的异常信息

以上是大体的思路，都是纯手工的，而rancher平台能把异常的event信息发送到ui界面

1. 先看对应资源的状态信息是否正常，比如pod创建不成功
    ````
    > kubectl get pods -A  
    NAMESPACE         NAME                                      READY   STATUS             RESTARTS   AGE    
    mymysql-p-rh8cs   mymysql-p-rh8cs-9cd965fd9-4vcln           0/1     Pending            0          5m50s
    ````
    status为pending，ready未就绪。
2. 查看该pod对应的更详细的信息，如event
    ```
    > kubectl describe pods -n mymysql-p-rh8cs mymysql-p-rh8cs-9cd965fd9-4vcln
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    Events:
      Type     Reason            Age        From               Message
      ----     ------            ----       ----               -------
      Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
      Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
    ```
    从这里我们可以看出状态暂停是因为调度失败，没有node可以调度,`Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.`.

3. 如果上面那一步还没有解决问题的话，则往往需要查看pod日志或者容器的日志
`kubectl logs -n ingress-nginx nginx-ingress-controller-5wct`
,如果只有一个容器，则可以不用-c container，如果有多个容器的话，则需要-c container，容器名称查看可以直接在`kubectl describe`
    ```
    > kubectl logs -n ingress-nginx nginx-ingress-controller-5wct8    
    -------------------------------------------------------------------------------
    NGINX Ingress controller
      Release:       nginx-0.35.0-rancher2
      Build:         git-ba9a25e23
      Repository:    https://github.com/rancher/ingress-nginx.git
      nginx version: nginx/1.19.2

    -------------------------------------------------------------------------------

    I0420 01:11:15.156309       8 flags.go:205] Watching for Ingress class: nginx
    W0420 01:11:15.156387       8 flags.go:210] Ingresses with an empty class will also be processed by this Ingress controllernginx
    W0420 01:11:15.156602       8 flags.go:252] SSL certificate chain completion is disabled (--enable-ssl-chain-completion=false)
    W0420 01:11:15.157096       8 client_config.go:552] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
    I0420 01:11:15.159120       8 main.go:231] Creating API client for https://10.43.0.1:443
    I0420 01:11:15.238096       8 main.go:275] Running in Kubernetes cluster version v1.17 (v1.17.17) - git (clean) commit f3abc15296f3a3f54e4ee42e830c61047b13895f - platform linux/amd64
    I0420 01:11:15.353651       8 main.go:87] Validated ingress-nginx/default-http-backend as the default backend.
    I0420 01:11:15.789693       8 main.go:105] SSL fake certificate created /etc/ingress-controller/ssl/default-fake-certificate.pem
    I0420 01:11:15.877107       8 nginx.go:263] Starting NGINX Ingress controller
    I0420 01:11:15.909830       8 event.go:278] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginx", Name:"nginx-configuration", UID:"46a70bd1-d6d0-4c48-9dc0-7aff315b0432", APIVersion:"v1", ResourceVersion:"514", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/nginx-configuration
    I0420 01:11:15.911364       8 event.go:278] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginx", Name:"tcp-services", UID:"b016eabe-0dec-4ab6-ac13-2c1e3eb5980e", APIVersion:"v1", ResourceVersion:"517", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/tcp-services
    I0420 01:11:15.911378       8 event.go:278] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginx", Name:"udp-services", UID:"3b80e1d3-0096-4a9a-a764-c4c729026425", APIVersion:"v1", ResourceVersion:"518", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/udp-services
    I0420 01:11:17.017313       8 event.go:278] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"grafana", Name:"grafana", UID:"2af1c546-ecc3-4cf9-8e99-5c5d1eb1711a", APIVersion:"networking.k8s.io/v1beta1", ResourceVersion:"2249", FieldPath:""}): type: 'Normal' reason: 'CREATE' Ingress grafana/grafana
    I0420 01:11:17.078433       8 leaderelection.go:242] attempting to acquire leader lease  ingress-nginx/ingress-controller-leader-nginx...
    I0420 01:11:17.078395       8 nginx.go:307] Starting NGINX process
    W0423 12:40:21.962741       8 controller.go:395] Service "ingress-nginx/default-http-backend" does not have any active Endpoint
    W0423 12:40:25.273797       8 controller.go:395] Service "ingress-nginx/default-http-backend" does not have any active Endpoint
    W0423 12:40:28.593468       8 controller.go:395] Service "ingress-nginx/default-http-backend" does not have any active Endpoint
    W0423 14:00:38.192983       8 controller.go:395] Service "ingress-nginx/default-http-backend" does not have any active Endpoint
    ```
