# python相关的知识点

#### python装饰器
[python装饰器的wraps作用](https://blog.csdn.net/liuskyter/article/details/80357647)
**双层封装器**
```
# 第一层
def redis_set(func):
    @wraps(func)
    def func_wrapper(self, *args, **kwargs):
        RedisHandler.cache_set(self.ip, {"timestamp": int(time.time()), "status": 1})
        ret = func(self, *args, **kwargs)
        RedisHandler.cache_set(self.ip, {"timestamp": int(time.time()), "status": 0})
        return ret
    return func_wrapper
    
# 第二层
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
 
 # 调用装饰器
 @redis_set
 @log()
 def remove_trunk_vlan(self, port, vlan, native=False):
 ```
#### 分布式celery
参考链接 https://segmentfault.com/a/1190000016082551

