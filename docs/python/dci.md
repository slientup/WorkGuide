# sdn项目
该项目是sdn设备的配置管理的南向接口，涉及的技术django+celery+restful api+cache(django自带缓存)

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


二，再比如，我们在网站放到服务器上正式运行后，DEBUG改为了 False，这样更安全，但是有时候发生错误我们不能看到错误详情，调试不方便，有没有办法处理好这两个事情呢？
普通访问者看到的是友好的报错信息
管理员看到的是错误详情，以便于修复 BUG
https://code.ziqiangxuetang.com/django/django-middleware.html
