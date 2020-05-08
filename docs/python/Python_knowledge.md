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
它们的区别在于def m3(n):  def m2(`cls`, n):  `staticmethod`不需要写上`cls` 跟普通方法一模一样

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

#### format格式化字符串
如果待格式化的字符串中含有`{}`，则可以通过`{{}}`的方式实现字符串中含有{}
```
"{} {}".format("hello", "world")   #  hello world
"{0} {{test}} {1}".format("hello", "world")   #  hello {test} world
```
####  迭代器、生成器、yield
参考链接：https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do  
**迭代器** 迭代器里面所有的值都保存在内存中 占内存空间
**生成器** 当我们需要某个值的，会立即生成，用完就会销毁 即需要某个值才生成 不占内存空间 但不能重复利用
当数据量大的时候，一定要使用生成器

```
>>> mylist = [x*x for x in range(3)]   这里是迭代器
>>> for i in mylist:
...    print(i)
0
1
4

>>> mygenerator = (x*x for x in range(3))  这里是生成器  仅仅是将[]替换成（）
>>> for i in mygenerator:
...    print(i)
0
1
4
```
`yield` is a keyword that is used like return, except the function will return a `generator`
eg：
```
>>> def createGenerator():
...    mylist = range(3)
...    for i in mylist:
...        yield i*i
...
>>> mygenerator = createGenerator()   # create a generator  
>>> print(mygenerator) # mygenerator is an object!
<generator object createGenerator at 0xb7555c34>
>>> for i in mygenerator:
...     print(i)
0
1
4
```
#### python中重载方法
python中没有重载方法，因为python中的方法变量数量可以是任意的，同时也可以指定默认值

#### python变量查找顺序
先从局部——全局的方式
本地作用域（Local）→当前作用域被嵌入的本地作用域（Enclosing locals）→全局/模块作用域（Global）→内置作用域（Built-in）

#### GIL锁
GIL是什么？全局解释锁，python设计之初为了线程安全而做的
在python多线程下，**每个线程的执行方式**：
1. 获取GIL
2. 执行代码知道sleep或者python虚拟机将其挂起
3. 释放GIL
**可见，某个线程想要执行，必须先拿到GIL，我们可以把GIL看作是“通行证”，并且在一个python进程中，GIL只有一个。拿不到通行证的线程，就不允许进入CPU执行**
GIL锁释放逻辑是：在python2.x里，GIL的释放逻辑是当前线程遇见`IO操作`或者`ticks计数达到100(很容易到达)`（ticks可以看作是python自身的一个计数器，专门做用于GIL，每次释放后归零，这个计数可以通过 sys.setcheckinterval 来调整），进行释放

IO型任务 `多线程`  cpu密集型任务 `多进程`







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

