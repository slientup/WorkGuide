# python相关的知识点

一 基础
---
#### 浅拷贝和深拷贝
- copy() 浅拷贝  浅拷贝是引用  
- deepcopy() 深拷贝 深拷贝完全新生成一个对象
```
import copy
a = [1, 2, 3, 4, ['a', 'b']]  #原始对象

b = a  #赋值，传对象的引用
c = copy.copy(a)  #对象拷贝，浅拷贝
d = copy.deepcopy(a)  #对象拷贝，深拷贝

a.append(5)  #修改对象a

```
#### is和==的区别
is是对比地址  ==是对比值

#### read,readline和readlines
- read 读取整个文件
- readline 一行读取到内存 使用生成器 **在量很大的情况下 使用这种方式处理文件**
- readlines 读取整个文件到一个迭代器中 供我们遍历

#### range and xrange
必须使用**xange**     
`xrange`是懒加载，当你需要某个数的时候才生成，而`range`是一次性生成， range(1,1000) 就会在内存里面生成1000个数

#### @staticmethod和@classmethod区别
```
    class A(object):
        def m1(self, n):
            print("self:", self)

        @classmethod
        def m2(cls, n):
            print("cls:", cls)

        @staticmethod
        def m3(n):
            pass
    a = A()
    a.m1(1) # self: <__main__.A object at 0x000001E596E41A90>
    A.m2(1) # cls: <class '__main__.A'>
    A.m3(1)
```
它们的区别在于def m3(n):  def m2(cls, n):  `staticmethod`不需要写上`cls` 跟普通方法一模一样

#### 可变类型的变量和不可变类型变量
`strings`, `tuples`, 和`numbers`是不可更改的对象(**类似java常量池**)，而 `list, dict, set` 等则是可以修改的对象 
举例
```
a = 1
def fun(a):
    a = 2
fun(a)
print a  # 1  没有改变
```
```
a = []
def fun(a):
    a.append(1)
fun(a)
print a  # [1]  改变了

```
参考链接：https://github.com/slientup/interview_python#1-python%E7%9A%84%E5%87%BD%E6%95%B0%E5%8F%82%E6%95%B0%E4%BC%A0%E9%80%92

#### class中特殊的方法
获取class对象运行相关的方法 在动态处理上非常有用
- type() 获取对象类型
- dir() 获取对象所有方法和属性
- getattr() 获取具体的某个属性
- hasattr() 判断某个属性是否存在
- isinstance() 判断当前对象类型与给定的是否一致

#### python中的变量_和__
- xx：公有变量
- `_xx`: 前置单下划线，私有化属性或方法 只是约定俗称
- `__xx`: 前置双下划线，私有化属性或者方法 强制的
```
class test(object):
  def __init__(self):
     self.x = 10
     self._x = 20
     self.__x = 30
t = test()
print(t.x) # 10
print(t._x) # 20
# print(t.__x) # AttributeError: 'test' object has no attribute '__x'
```

#### @property
将类中的方法变成属性，通过obeject.方法名的方式调用，而不是()的方式调用
```
class Student(object):
    @property
    def birth(self):
        return self._birth
    @birth.setter
    def birth(self, value):
        self._birth = value
    
    student = Student()
    student.birth=10
    student.birth   <10>
```
参考 https://www.liaoxuefeng.com/wiki/897692888725344/923030547069856




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

