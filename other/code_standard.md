### 代码规范
[guide哥整理的代码规范](https://snailclimb.gitee.io/javaguide/#/docs/java/Java%E7%BC%96%E7%A8%8B%E8%A7%84%E8%8C%83)

看过的项目中代码规范示例：  
- python : netos  
- java : apollo  
#### 整体原则
代码层级结构要明显 逻辑清晰  责任分明 分组管理  好的代码看名字就能看出来

- 控制器 只应该有整体流程的调用 后台service的处理逻辑不要包含在这里面
- 重复的代码就要抽象出来，进行再次封装后调用
  `案例1`：通过装饰器进行日志封装
  ```
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
      
    @log()  //调用方法
    def destroy_port(self,):
  ```
  `案例2`:对response进行封装
 
 ```
 class APIResponse(response.Response):
    def __init__(self, data=list(), status=True,
                 message="", extra=None, **kwargs):
        if not message:
            message = "success" if status else "failure"
        _data = {
            "data": data,
            "message": str(message),
            "status": status,
        }
        if isinstance(extra, dict):
            _data.update(extra)
        super(APIResponse, self).__init__(data=_data, status=200, **kwargs)

    def render(self):
        return super(APIResponse, self).render()
 ```
  
  
  
  




