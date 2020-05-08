# sdn项目
该项目是sdn设备的配置管理的南向接口，涉及的技术django+celery+restful api+cache(django自带缓存)，haproxy+keepalived高可用部署

一 项目模块
---

- `agent`   获取设备信息相关的操作放在该模块(用于维护) 
- `ansible` 通过ansible做自动化批量发布
- `api`     提供给第三方调用 这个第三方是dashboard或者其他应用程序
- `bin`      采集任务通过bin的方式 手动执行
- `conf`      保存线上环境的配置信息 nginx supervisor ini
- `inventory`  该模块里面存放的是采集设备信息的task  存储的是device信息
- `libs`      公用的底层连接库 snmp ssh netconf等
- `middleware` 中间件
- `overlay`   overlay是业务逻辑代码 最终调用southbound接口下发配置 service的概念
- `pool`      跟地址池相关的模块
- `scripts`    监控检测脚本
- `SDNController` celery log等系统配置
- `southbound`   对南向设备进行下发配置的时候会调用该接口(下发配置)
- `surface`   页面展示模块
-  装饰器 常量

业务逻辑是：`api——>overlay——>southbound——>netconf`
其他模块都是为其服务的

二 模块详细分析
---
### API模块
API模块使用的是Django REST framework框架 该框架主要提供`APIView`与`Viewsets`这两种方式提供api接口  
Django REST framework的知识点 见github官网  
django 权限相关的知识点：https://www.chenshaowen.com/blog/permissions-of-django-rest-framework.html 

#### 两类权限校验    
`IsInWhiteList`(白名单)  `IsNetManager`(网络组)  
白名单 主要是针对无法认证的用户 即合法api接口调用者   
网络组 主要是网页判断用户是否是网络组 只有网络组的用户才拥有权限下发配置
```
from constance import config
    def has_permission(self, request, view):
        ip_addr = request.META["REMOTE_ADDR"]
        white_list_ips = config.white_list_ips
        if not request.user.is_authenticated():
            if white_list_ips and request.method not in permissions.SAFE_METHODS:
                for item in white_list_ips.split(","):
                    if IPAddress(ip_addr) in IPNetwork(item):
                        return True
                return False
        return True
```
通过`django_constance`来管理动态的配置变量 参考链接：https://www.jianshu.com/p/22a991582d54     
`SAFE_METHODS = ('GET', 'HEAD', 'OPTIONS')`  安全的方法就是可读的方法

#### @detail_route 
在django_rest_framework 定义额外的路由信息 参考链接 https://juejin.im/post/5a991807518825558a060a77
```
    @detail_route(methods=['post'])
    def set_password(self, request, pk=None):
        user = self.get_object()
        serializer = PasswordSerializer(data=request.data)
```
访问url `users/{pk}/set_password` 重点是这里的**set_password**

#### rest中method对应关系
- 'get': 'list',
- 'post': 'create'
- 'get': 'retrieve',
- 'put': 'update',
- 'patch': 'partial_update',
- 'delete': 'destroy'

#### raise Exception("no ip resources")  抛出异常

#### rest处理异常
```
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'api.utils.exceptions.custom_exception_handler',
    'DEFAULT_PAGINATION_CLASS': 'api.decorators.CustomPagination',
    'PAGE_SIZE': 20
}
```
### bin模块
该模块是通过命令的方式运行采集任务
### domain模块
不太清楚domain模块的作用
### inventory模块
该模块通过定时任务获取设备的资源信息 如lldp信息 硬件资源信息
由于有多个终端的存在，这里做一个工厂方法，根据设备型号和任务类型 返回对应的类
```
def get_collect_factory(device, collector):
    package = __import__("agent.collectors")
    module = getattr(package, "collectors")    # 对象的属性值 CiscoHardWare  hardWare
    sub_module = getattr(module, collector)
    return getattr(sub_module, "{0}{1}".format(device.manufacturer.capitalize(), COLLECT_MAPPING[collector]))
    
@shared_task(ignore_result=True)
@agent_log()
def collect_check_sync_task(d):   
    return get_collect_factory(d, "check_sync")(d).check()  //初始化调用
```
调用逻辑：定时任务——inventory——agent.collector——southbound
### middleware模块
主要解决超级用户访问报错的时候，直接返回报错信息，而用户访问报错的时候，返回友好的报错内容
https://code.ziqiangxuetang.com/django/django-middleware.html
```
class UserBasedExceptionMiddleware(object):
    def process_exception(self, request, exception):
        if request.user.is_superuser or request.META.get('REMOTE_ADDR') in getattr(settings, "INTERNAL_IPS"):
            return technical_500_response(request, *sys.exc_info())
```

### overlay模块
在模型设计中所有类都继承`TimestampedModel`  这是非常好的设计
```
class TimestampedModel(models.Model):
    objects = MyQuerySet.as_manager()
    created_at = models.DateTimeField(auto_now_add=True, null=True)
    updated_at = models.DateTimeField(auto_now=True, null=True)

    class Meta:
        abstract = True 
```
模型中增加property字段 像访问属性点的方式 `.` 访问字段
```
class SUBNet(TimestampedModel):
    uuid = models.CharField(max_length=150, unique=True)
    network = models.ForeignKey(Network)
    name = models.CharField(max_length=100)
    cidr = models.CharField(max_length=100, unique=True)
    gateway = models.GenericIPAddressField(null=True)
    dhcp = models.BooleanField(default=True)
    shared = models.BooleanField(default=False)

    def __unicode__(self):
        return self.name

    @property   
    def mask(self):   
        return self.cidr.split("/")[1]

    @property
    def net(self):
        return self.cidr.split("/")[0]

    @property
    def router_interfaces(self):
        return self.routerinterface_set.all()
      
    subnet_obj.mask 访问mask属性
    
```
### script模块 
该模块针对的是supervisor做监控检测的 通过调用对应的api接口 https://www.cnblogs.com/halleluyah/p/9674584.html

### southbound模块
这模块都是网络设备下发相关的命令封装，然后再调用netconf接口下发配置,跨系统的位置必须封装log

```
    @redis_set
    @log()
    def create_external2_bgp(self, vrf, bgp_asn, peer_ip, template, remote_bgp_asn):
        cmd = ["conf t",
               "router bgp %s" % bgp_asn,
               "vrf %s" % vrf,
               "neighbor %s" % peer_ip,
               "inherit peer %s" % template,
               "remote-as %s" % remote_bgp_asn]
        self.nc.send_commands(cmd)
        return cmd
  
  这是日志的装饰器
  def log(param='log'):
    def decorator(func):
        @wraps(func)
        def func_wrapper(self, *args, **kwargs):
            try:
                ret = func(self, *args, **kwargs)
                southbound.info("%s_%s_%s" % (func.__name__, self.ip, ret))  
                return ret
            except Exception, e:
                southbound.error("%s_%s_%s" % (func.__name__, self.ip, e))
                raise Exception(e)
        return func_wrapper
    return decorator
```
日志进行封装的时候，必须涵盖有用信息，比如调用的方法模块，访问url 访问内容 结果状态等信息

### utils模块
该模块是对http调用外部系统的再次封装上实现 在对httpclient进行封装的时候,将`url`中的`地址`和`uri`分开会更好 

对于本地变量常用`_`
```
class Netranger(Client):
    _osg_token = None
    @classmethod
    def _rest_call(cls, url, method="GET", **kwargs):
        if 'headers' not in kwargs:
            kwargs['headers'] = dict()
        kwargs['headers']['Content-Type'] = 'application/json'
        kwargs['headers']['Accept'] = 'application/json'
        return super(Netranger, cls)._rest_call(url, method, **kwargs)
```  
### 日志流
将日志写到本地———>logagent——————>kafka——————>es 还可以更好  topic `ops.net.sdn`
### 待解决问题
1. celery是不是只使用了一个队列？
任务队列分配信息在`celery_config`中
```
    def route_for_task(self, task, args=None, kwargs=None):
        if task.startswith("inventory.tasks.collect"):
            routing_key, exchange = "device", "collector"
        else:
            exchange = "port"
            if task == "inventory.tasks.create_port_task":
                routing_key = "device%s" % args[0].ip[-1]
            elif task == "inventory.tasks.destroy_port_task":
                routing_key = "device%s" % args[0].ip[-1]
            elif task == "overlay.tasks.create_router_interface_task":
                routing_key = "device%s" % args[0].ip[-1]
            elif task == "overlay.tasks.destory_router_interface_task":
                routing_key = "device%s" % args[0].ip[-1]
            else:
                routing_key = "port"
        return {
            "exchange": exchange,
            "exchange_type": "direct",
            "routing_key": routing_key
        }
  ```
 2. 如何监控各个组件的好坏？比如说网站是否可用  
 通过hickwall的`remote.snmp.socket`采集器测试端口是否可达
 
  












