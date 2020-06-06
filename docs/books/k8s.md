# 深入剖析Kubernetes
容器技术兴起源于Paas技术的普及，
docker公司发布的docker项目具有里程碑的意义，
docker项目通过容器镜像，解决了**应用打包**这个根本性难题
容器本身没有价值，有价值的是**容器编排**，容器技术生态爆发了一场关于容器编排的战争，最终以kubernetes项目和CNCF社区的胜利而告终。


## 容器技术概念入门篇

### 从进程说开去

沙盒技术：
沙盒就是能够像一个集装箱一样，把你的应用“装”起来的技术，这样，应用与应用之间，就因为有了**边界而不至于相互干扰**；而被装进集装箱的应用，
也可以**被方便地搬来搬去**，这不就是 PaaS 最理想的状态嘛。

首先，操作系统从“程序”中发现输入数据保存在一个文件中，所以这些数据就会被加载到内存中待命。同时，操作系统又读取到了计算加法的指令，
这时，它就需要指示 CPU 完成加法操作。而 CPU 与内存协作进行加法计算，又会使用寄存器存放数值、内存堆栈保存执行的命令和变量。
同时，计算机里还有被打开的文件，以及各种各样的 I/O 设备在不断地调用中修改自己的状态。
就这样，一旦“程序”被执行起来，它就**从磁盘上的二进制文件**，变成了计算机内存中的数据、寄存器里的值、堆栈中的指令、被打开的文件，以及各种设备的
状态信息的一个集合。**像这样一个程序运行起来后的计算机执行环境的总和，就是我们今天的主角：进程**


对于进程来说，它的静态表现就是程序，平常都是安安静静地待在磁盘上，而一旦运行起来，它就变成了计算机里的**数据和状态**总和，这就是它的动态表现。


容器技术的**核心功能**:通过约束和修改进程的动态表现，从而为其创造出一个**边界**

而`docker`主要通过`cgroups`技术和`namespace`技术来实现，其中`Cgroups`技术是用来制造约束的主要手段,`Namespace`技术则是用来修改进程视图的主要方法

```linux
$ docker run -it busybox /bin/sh
请帮我启动一个容器，在容器里执行 /bin/sh，并且给我分配一个命令行终端跟这个容器交互
/ # ps
PID  USER   TIME COMMAND
  1 root   0:00 /bin/sh
  10 root   0:00 ps
```
从上面信息我们可以看到docker容器最开始执行进程是`/bin/sh` 即第1号进程(PID=1),  它究竟是怎么做到的啦？
当我们在宿主机中运行`/bin/sh`的时候，操作系统会为它分配一个进程编号，比如PID=100，这个编号是进程的唯一标识，而`docker`会做障眼法，让`pid=100`看不到
之前的99个PID进程，让它误以为自己是1号进程

linux是怎么做到的啦？这就是**Namespace机制** 
```linux
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
```
当我们用 clone() 系统调用创建一个新进程时，就可以在参数中指定 CLONE_NEWPID 参数,新创建的这个进程将会“看到”一个全新的进程空间，在这个进程空间里，
它的 PID 是 1

而除了我们刚刚用到的 PID Namespace，Linux 操作系统还提供了 Mount、UTS、IPC、Network 和 User 这些 Namespace，用来对各种不同的进程上下文进行“障眼法”操作。
比如，`Mount Namespace`，用于让被隔离进程只看到当前 Namespace 里的挂载点信息；`Network Namespace`，用于让被隔离进程看到当前 `Namespace` 里的网络设备和配置.

**结论：** 容器，其实是一种特殊的进程而已
![容器](https://static001.geekbang.org/resource/image/d1/96/d1bb34cda8744514ba4c233435bf4e96.jpg)

### 隔离与限制
`Namespace`技术实际上修改了应用进程看待整个计算机“视图”，即它的“视线”被操作系统做了限制，只能“看到”某些`指定的内容`.
`docker`容器本身只是一个特殊的进程，它不占用性能损耗，所以在paas上面进行更细粒度的管理平台上大行其道，但因为是一个`特殊的进程`，所以最主要的问题
是**隔离得不彻底**

**首先，既然容器只是运行在宿主机上的一种特殊的进程，那么多个容器之间使用的就还是同一个宿主机的操作系统内核**

**其次，在 Linux 内核中，有很多资源和对象是不能被 Namespace 化的，最典型的例子就是：时间**

如果你的容器中的程序使用 settimeofday(2) 系统调用修改了时间，整个宿主机的时间都会被随之修改，这显然不符合用户的预期，所以"什么能做，什么不能做"就
必须考虑下。

**我们会遇到这么一个问题？**

虽然容器内的第 `1`号进程在“障眼法”的干扰下只能看到容器里的情况，但是宿主机上，它作为第 `100` 号进程与其他所有进程之间依然是**平等的竞争关系。**
这就意味着，虽然第`100`号进程表面上被隔离了起来，但是它所能够使用到的**资源（比如 CPU、内存）**，却是可以随时被宿主机上的其他进程（或者其他容器）占用的。
当然，这个`100`号进程自己也可能把**所有资源吃光**。这些情况，显然都不是一个“沙盒”应该表现出来的合理行为.

而 **Linux Cgroups (Linux Control Group) 就是 Linux 内核中用来为进程设置资源限制的一个重要功能** 就能解决这个问题，它主要的作用,就是限制一个进程组能够使用的资源上限，包括
`cpu 内存 磁盘 网络带宽`等等

通过`/sys/fs/cgroup`下面 cpuset、cpu、 memory 的子目录等信息分别对不同资源进行限制

比如 需要对cpu资源做限制
`cfs_period` 和 `cfs_quota` 这样的关键词。这两个参数需要组合使用，可以用来限制进程在长度为 `cfs_period` 的一段时间内，只能被分配到
总量为 `cfs_quota` 的 CPU 时间.

```linux
我们可以通过查看 container 目录下的文件，看到 container 控制组里的 `CPU quota` 还没有任何限制（即：-1），`CPU period` 则是默认的 100  ms（100000  us）

$ cat /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us 
-1
$ cat /sys/fs/cgroup/cpu/container/cpu.cfs_period_us 
100000

我们可以做一个限制 比如向 container 组里的 cfs_quota 文件写入 20 ms（20000 us）
echo 20000 > /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us
```
它意味着在每 `100ms`的时间里，被该控制组限制的进程只能使用 `20ms` 的 CPU 时间，也就是说这个进程只能使用到 `20%` 的 CPU 带宽

docker中启动cpu限制
```
$ docker run -it --cpu-period=100000 --cpu-quota=20000 ubuntu /bin/bash
```

一个正在运行的 Docker 容器，其实就是一个启用了**多个 Linux Namespace 的应用进程**，而这个进程能够使用的资源量，则受`Cgroups`配置的限制

**容器是一个单进程模型**

 但实际中，跟Namespace 的情况类似，`Cgroups`对资源的限制能力也有很多不完善的地方，被提及最多的自然是`/proc`文件系统的问题
 
 比如：你如果在容器里执行 top 指令，就会发现，它显示的信息居然是宿主机的CPU 和内存数据，而不是当前容器的数据.?Linux 下的 /proc 目录存储的是记录当前内核运行状态的一系列特殊文件，用户可以通过访问这些文件，查看系统以及当前正在运行的进程的信息，比如
 CPU 使用情况、内存占用率等，这些文件也是 top 指令查看系统信息的主要数据来源。/proc 文件系统并不知道用户通过 Cgroups 给这个容器做了什么样的资源限制，即：/proc 文件系统不了解 Cgroups 限制的存在

**深入理解容器镜像**
 默认情况下当我们进入一个“容器”，执行一下 ls 指令的话，我们就会发现一个有趣的现象： `/tmp`目录下的内容跟宿主机的内容是一样的,因为**Mount Namespace 修改的，是容器进程对文件系统“挂载点”的认知**  只有在“挂载”这个操作发生之后，进程的视图才会被改变。而在此之前，新创建的容器会直接继承宿主机的各个挂载点

这就是 `Mount Namespace` 跟其他 `Namespace` 的使用略有不同的地方：它对容器进程视图的改变，一定是伴随着挂载操作（`mount`）才能生效

为了能够让容器的这个根目录看起来更“真实”，我们一般会在这个容器的根目录下挂载一个完整操作系统的文件系统，比如 Ubuntu16.04 的 ISO。这样，在容器启动之后，我们在容器里通过执行 "ls /" 查看根目录下的内容，就是 Ubuntu 16.04 的所有目录和文件.

**而这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“容器镜像”。它还有一个更为专业的名字，叫作：rootfs（根文件系统）**

对于`docker`来说,最核心的原理：
- 启用 Linux Namespace 配置；  为进程进行隔离
- 设置指定的 Cgroups 参数；   为进程分配资源利用率
- 切换进程的根目录（Change Root）  容器的文件系统

**重点：** 需要明确的是，`rootfs`只是一个操作系统所包含的文件、配置和目录，**并不包括操作系统内核**。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像;
所以说，·rootfs·只包括了操作系统的“躯壳”，并没有包括操作系统的“灵魂”

那么，对于容器来说，这个操作系统的“灵魂”又在哪里呢？

实际上，同一台机器上的所有容器，都共享**宿主机操作系统的内核。也就是对内核的参数的修改，就会影响到所有的容器，牵一发而动全身。**

容器的**一致性**
由于`rootfs`里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着，应用以及它运行所需要的所有依赖，都被封装在了一起,对一个应用来说，
操作系统本身才是它运行所需要的最完整的**依赖库** , 无论在本地、云端，还是在一台任何地方的机器上，用户只需要解压打包好的容器镜像就可以运行

**这种深入到操作系统级别的运行环境一致性，打通了应用在本地开发和远端执行环境之间难以逾越的鸿沟**  

**容器镜像将会成为未来软件的主流发布方式**

难道我每开发一个应用，或者升级一下现有的应用，都要重复制作一次 rootfs 吗？
```
Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs
```
```
$ docker run -d ubuntu:latest sleep 3600  

这个所谓的“镜像”，实际上就是一个 Ubuntu 操作系统的 rootfs，实际上就是一个 Ubuntu 操作系统的 rootfs，它的内容是 Ubuntu 操作系统的所有文件和目录

$ docker image inspect ubuntu:latest  
...
     "RootFS": {
      "Type": "layers",
      "Layers": [                 >>>> layer 这五个层就是增量rootfs
        "sha256:f49017d4d5ce9c0f544c...",
        "sha256:8f2b771487e9d6354080...",
        "sha256:ccd4d61916aaa2159429...",
        "sha256:c01d74f99de40e097c73...",
        "sha256:268a067217b5fe78e000..."
      ]
    }
```
最终挂载点：是由基础和增量合并生成 使用多个增量 rootfs 联合挂载一个完整 rootfs 的方案，这就是容器镜像中“层”的概念

```
$ cat /sys/fs/aufs/si_972c6d361e6b32ba/br[0-9]*
/var/lib/docker/aufs/diff/6e3be5d2ecccae7cc...=rw
/var/lib/docker/aufs/diff/6e3be5d2ecccae7cc...-init=ro+wh
/var/lib/docker/aufs/diff/32e8e20064858c0f2...=ro+wh
/var/lib/docker/aufs/diff/2b8858809bce62e62...=ro+wh
/var/lib/docker/aufs/diff/20707dce8efc0d267...=ro+wh
/var/lib/docker/aufs/diff/72b0744e06247c7d0...=ro+wh
/var/lib/docker/aufs/diff/a524a729adadedb90...=ro+wh
```
容器镜像文件rootfs有三部分组成：
![容器镜像三层](https://static001.geekbang.org/resource/image/8a/5f/8a7b5cfabaab2d877a1d4566961edd5f.png)

- 只读层：容器镜像本身内容的层
- 可读可写层：容器层，用于在容器内部做修改的一层
上层的读写层通常也叫容器层，下面的只读层称为镜像层，所有的增删查改只会作用在容器层，相同的文件上层会覆盖掉下层。比如修改文件，首先会从上到下查找
有没有这个文件，找到，就复制到容器层中，修改，修改的结果就会作用到下层的文件，这种方式称为`copy-on-write`

**可读可写层：**
它是这个容器的 rootfs 最上面的一层（6e3be5d2ecccae7cc），它的挂载方式为：rw，即 read write。在没有写入文件之前，这个目录是空的。而一旦在容器里做了写操作，你修改产生的内容就会以增量的方式出现在这个层中。

可是，你有没有想到这样一个问题：如果我现在要做的，是删除只读层里的一个文件呢？为了实现这样的删除操作，AuFS 会在可读写层创建一个 whiteout 文件，把只读层里的文件“遮挡”起来。

比如，你要删除只读层里一个名叫 foo 的文件，那么这个删除操作实际上是在可读写层创建了一个名叫.wh.foo 的文件。
这样，当这两个层被联合挂载之后，foo 文件就会被.wh.foo 文件“遮挡”起来，“消失”了。这个功能，就是“ro+wh”的挂载方式，即只读 +whiteout 的含义。我喜欢把 whiteout 形象地翻译为：“白障”。

所以，最上面这个可读写层的作用，就是专门用来存放你修改 rootfs 后产生的增量，无论是增、删、改，都发生在这里。而当我们使用完了这个被修改过的容器之后，还可以使用 `docker commit` 和 `push` 指令，保存这个被修改过的可读写层，并上传到 `Docker Hub` 上，供其他人使用；而与此同时，原先的只读层里的内容则不会有任何变化。这，就是增量 rootfs 的好处
```
容器镜像将会成为未来软件的主流发布方式。
```
这种分层增量的思想与`git`非常像

### 重新认识docker容器
```linux
# 使用官方提供的Python开发镜像作为基础镜像
FROM python:2.7-slim

# 将工作目录切换为/app
WORKDIR /app

# 将当前目录下的所有内容复制到/app下
ADD . /app

# 使用pip命令安装这个应用所需要的依赖
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 允许外界访问容器的80端口
EXPOSE 80

# 设置环境变量
ENV NAME World

# 设置容器进程为：python app.py，即：这个Python应用的启动命令
CMD ["python", "app.py"]
```
**Dockerfile 的设计思想，是使用一些标准的原语（即大写高亮的词语），描述我们所要构建的 Docker 镜像。并且这些原语，都是按顺序处理的**

**Dockerfile 中的每个原语执行后，都会生成一个对应的镜像层**

容器镜像上传
``` 
$ docker tag helloworld geektime/helloworld:v1

$ docker push geektime/helloworld:v1
```
还可以使用` docker commit `指令，把一个正在运行的容器，直接提交为一个镜像
```
$ docker exec -it 4ddf4638572d /bin/sh
# 在容器内部新建了一个文件
root@4ddf4638572d:/app# touch test.txt
root@4ddf4638572d:/app# exit

#将这个新建的文件提交到镜像中保存
$ docker commit 4ddf4638572d geektime/helloworld:v2
```
查看容器的进程ID
```
$ docker inspect --format '{{ .State.Pid }}'  4ddf4638572d
25686
```
你可以通过查看宿主机的 proc 文件，看到这个 `25686`进程的所有 Namespace 对应的文件
```
$ ls -l  /proc/25686/ns
total 0
lrwxrwxrwx 1 root root 0 Aug 13 14:05 cgroup -> cgroup:[4026531835]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 ipc -> ipc:[4026532278]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 mnt -> mnt:[4026532276]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 net -> net:[4026532281]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 pid -> pid:[4026532279]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 pid_for_children -> pid:[4026532279]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 uts -> uts:[4026532277]
```
一个进程，可以选择加入到某个进程已有的 Namespace 当中，从而达到“进入”这个进程所在容器的目的，这正是 `docker exec`的实现原理


`docker commit`的原理：
实际上就是在容器运行起来后，把最上层的“可读写层”，加上原先容器镜像的只读层，打包组成了一个新的镜像。当然，下面这些只读层在宿主机上是共享的，不会占用额外的空间。

而由于使用了联合文件系统，你在容器里对镜像 rootfs 所做的任何修改，都会被操作系统先复制到这个可读写层，然后再修改。这就是所谓的：`Copy-on-Write`

而正如前所说，`Init`层的存在，就是为了避免你执行`docker commit` 时，把 Docker 自己对 `/etc/hosts`等文件做的修改，也一起提交掉，init层存放的具有本地属性的信息。

我们需要考虑这样两个问题
- 容器里进程新建的文件，怎么才能让宿主机获取到？
- 宿主机上的文件和目录，怎么才能让容器里的进程访问到？
这正是`Docker Volume`要解决的问题：Volume 机制，允许你将宿主机上指定的目录或者文件，挂载到容器里面进行读取和修改操作

**容器进程概念**

是 Docker 创建的一个容器初始化进程 (dockerinit)，而不是应用进程 (ENTRYPOINT + CMD)。dockerinit 会负责完成**根目录的准备、挂载设备和目录、配置 hostname 等一系列需要在容器内进行的初始化操作**。最后，它通过 execv() 系统调用，让应用进程取代自己，成为容器里的 PID=1 的进程。

挂载技术：就是Linux的`绑定挂载（bind mount）机制`。它的主要作用就是，允许你将一个目录或者文件，而不是整个设备，挂载到一个指定的目录上.

绑定挂载实际上是一个`inode`替换的过程。在 Linux 操作系统中，`inode`可以理解为存放文件内容的“对象”，而 `dentry`，也叫目录项，就是访问这个`inode`
所使用的“指针”
![](https://static001.geekbang.org/resource/image/95/c6/95c957b3c2813bb70eb784b8d1daedc6.png)
正如上图所示，`mount --bind /home /test`，会将 `/home` 挂载到 `/test` 上。其实相当于将 `/test` 的 dentry，重定向到了 `/home` 的 `inode`。这样当我们修改 `/test` 目录时，实际修改的是 `/home` 目录的 `inode`。这也就是为何，一旦执行 `umount` 命令，`/test` 目录原先的内容就会恢复：因为修改真正发生在的，是 /home 目录里

这个 /test 目录里的内容，既然挂载在容器 rootfs 的可读写层，它会不会被 docker commit 提交掉呢？

这个原因其实我们前面已经提到过。容器的镜像操作，比如 docker commit，都是发生在宿主机空间的。而由于 Mount Namespace 的隔离作用，宿主机并不知道这个绑定挂载的存在。所以，在宿主机看来，容器中可读写层的 /test 目录（/var/lib/docker/aufs/mnt/[可读写层 ID]/test），始终是空的

![docker容器全景图](https://static001.geekbang.org/resource/image/31/e5/3116751445d182687ce496f2825117e5.jpg)
这个容器进程“python app.py”，运行在由`Linux Namespace`和 `Cgroups`构成的**隔离环境**里；而它运行所需要的各种文件，比如 python，app.py，以及整个操作系统文件，则由多个联合挂载在一起的`rootfs`层提供.

**k8s**

作为一名开发者，我并不关心容器运行时的差异。因为，在整个“开发 - 测试 - 发布”的流程中，真正承载着容器信息进行传递的，是**容器镜像**，而不是容器运行时

**首先，Kubernetes 项目要解决的问题是什么？**
编排？调度？容器云？还是集群管理？
![k8s架构](https://static001.geekbang.org/resource/image/8e/67/8ee9f2fa987eccb490cfaa91c6484f67.png)
`kubelet`主要负责同容器运行时（比如 Docker 项目）打交道,而这个交互所依赖的，是一个称作 CRI（Container Runtime Interface）的远程调用接口，这个接口定义了容器运行时的各项核心操作.
另一个重要功能，则是调用网络插件和存储插件为容器配置网络和持久化存储。这两个插件与 kubelet 进行交互的接口，分别是 `CNI`（Container Networking Interface）和 `CSI`（Container Storage Interface）

解决**思路的不一样的**地方：
从一开始，Kubernetes 项目就没有像同时期的各种“容器云”项目那样，把 Docker 作为整个架构的核心，而**仅仅把它作为最底层的一个容器运行时实现**
它的核心解决的问题：**各种任务的关系**，运行在大规模集群中的各种任务之间，实际上存在着各种各样的关系。这些关系的处理，才是作业编排和管理系统最困难的地方










































 
 
 









































