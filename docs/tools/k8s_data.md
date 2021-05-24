#### 存储
> 存储是程序中很重要的一块的,针对不同的应用场景，k8s提供了不同的存储类型

##### secret
> secret解决敏感数据问题，如密码，token，密钥等
先定义secret，**存储在etcd中**，后再通过volume的方式调用

##### configMap
> 许多应用程序会从配置文件、命令行参数或环境变量中读取配置信息，而configMap就是解决配置文件的问题，我们不可能修改一个配置机重新打包镜像.

类似传统的配置中心，同样这类问题也可以通过配置中心来解决
也是**先定义configmap，存储在etcd中**，然后在volume中调用。


##### volume
存储卷分为临时卷，本地卷，持久化卷，在k8s中卷的类型很多，不同的类型拥有不同的生命周期，举几个例子。

**emptyDir** 

空目录，**只在节点本地使用**，一旦Pod被删除，则这个存储卷也会随之删除；所以，emptyDir这种存储卷不是做为持久化而设计的，可以做为临时目录，按需创建. 生命周期与pod一致.

**gitRepo**

在Pod创建时，会自动连接至Git仓库（需要确保宿主机有git命令，它是基于宿主机来驱动的。），**将Git仓库里的代码克隆至宿主机的某个目录，并把这个目录做为存储卷挂载至Pod上**,实质上`gitRepo`是建立在`emptyDir`之上的。`gitRepo`本质上也是`emptyDir`.

**hostPath**

宿主机路径，将**宿主机文件系统上的路径**做为存储卷挂载至`Pod`容器中。Pod删除后，此存储卷不会被删除，数据不会丢失. **本地卷 生命周期永久**，这与docker中的类似了。

**nfs**

nfs卷允许将现有的`NFS`（网络文件系统）共享挂载到您的容器中。不像 emptyDir，当删除 Pod 时，**nfs 卷的内容被保留**，卷仅仅是被卸载。这意味着 NFS 卷可以预填充数据，并且可以在 pod 之间“切换”数据。 NFS 可以被多个写入者同时挂载。(这种就是远端存储了)

**持久化volume**

持久化卷本意就是存储的内容是可以持久化的，主要是远端持久化，即存储的介质源头是分布式存储。


![](https://files.mdnice.com/user/4251/79dd97eb-a02c-47ac-958e-6f12a12f6726.png)

使用PVC的流程(涉及到几个概念)：

- **配置存储空间** --- 由存储管理员配置存储设备（如NFS，iSCSI，Ceph RBD，Glusterfs），并且划割好了很多可被独立使用的存储空间；

- **定义PV** --- K8S集群管理员将配置好的那些存储空间引入至K8S集群中，定义成PV （Persistent Volume，持久化卷）；

- **定义PVC** --- K8S用户在创建Pod时如果要用到PVC时，必须先创建PVC（在K8S集群中找一个能符合条件的存储卷PV来用）。注意：PV和PVC是一一对应关系，一旦某个PV被一个PVC占用，那么这个PV就不能再被其他PVC占用，被占用的PV的状态会显示为Bound。PVC创建以后，就相当于一个存储卷，可以被多个 Pod所使用。
- **使用PVC** --- 在Pod中去使用PVC，如果符合PVC条件的PV不存在，而这时去使用这个PVC，则Pod这时会显示Pending（挂起）状态。


但我们使用volume的时候，很多时候是与`stateful`的应用搭配使用，当这类应用需要做集群的时候，我们往往就需要使用动态分配存储了，因为你无法事先数量多少，而且也经常会变化。

更多的是使用下面两个概念

** StorageClass**  动态存储类

** volumeClaimTemplates**  pvc模板，当有需要的时候，就向动态存储拿一份。

案例：
```
spec:
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: gcr.io/google_containers/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
      annotations:
        volume.beta.kubernetes.io/storage-class: anything
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```








