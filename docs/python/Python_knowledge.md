# python相关的知识点
[基础](#基础)
  * [1 浅拷贝和深拷贝](#1-浅拷贝和深拷贝)
  * [2 is和==的区别](#2-is和==的区别)
  * [3 read,readline和readlines](#3-read,readline和readlines)
  * [4 range和xrange](#4-range和xrange)
  * [5 @staticmethod和@classmethod区别](#5-@staticmethod和@classmethod区别)
  * [6 可变类型的变量和不可变类型变量](#6-可变类型的变量和不可变类型变量)
  * [7 class中特殊的方法](#7-class中特殊的方法)
  * [8 python中的变量_和__](#8-python中的变量_和__)
  * [9 @property](#9-@property)
  * [10 format格式化字符串](#10-format格式化字符串)
  * [11 迭代器、生成器和yield](#11-迭代器、生成器和yield)
  * [12 python中重载方法](#12-python中重载方法)
  * [13 python变量查找顺序](#13-python变量查找顺序)
  * [14 GIL锁](#14-GIL锁)
  * [15 __new__和__init__的区别](#15-__new__和__init__的区别)
  * [16 单例模式](#16-单例模式)
  * [17 lambda函数](#17-lambda函数)
  * [18 高阶函数](#18-高阶函数)
  * [19 python垃圾回收机制](#19-python垃圾回收机制)
  * [20 super()方法](#20-super()方法)
  * [21 MRO顺序](#21-MRO顺序)
  * [22 装饰器wraps的作用](#22-装饰器wraps的作用)
  * [23 双层装饰器](#23-双层装饰器)
  
  
一 基础
---
#### 1 浅拷贝和深拷贝
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
#### 2 is和==的区别
is是对比地址  ==是对比值

#### 3 read,readline和readlines
- read 读取整个文件
- readline 一行读取到内存 使用生成器 **在量很大的情况下 使用这种方式处理文件**
- readlines 读取整个文件到一个迭代器中 供我们遍历

#### 4 range和xrange
必须使用**xange**     
`xrange`是懒加载，当你需要某个数的时候才生成，而`range`是一次性生成， range(1,1000) 就会在内存里面生成1000个数

#### 5 @staticmethod和@classmethod区别
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

#### 6 可变类型的变量和不可变类型变量
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

#### 7 class中特殊的方法
获取class对象运行相关的方法 在动态处理上非常有用
- type() 获取对象类型
- dir() 获取对象所有方法和属性
- getattr() 获取具体的某个属性
- hasattr() 判断某个属性是否存在
- isinstance() 判断当前对象类型与给定的是否一致

#### 8 python中的变量_和__
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

#### 9 @property
将类中的方法变成属性，通过obeject.方法名的方式调用，而不是()的方式调用 当对某个字段需要格式化处理的时候也可以使用property
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

#### 10 format格式化字符串
如果待格式化的字符串中含有`{}`，则可以通过`{{}}`的方式实现字符串中含有{}
```
"{} {}".format("hello", "world")   #  hello world
"{0} {{test}} {1}".format("hello", "world")   #  hello {test} world
```
#### 11 迭代器、生成器和yield
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
...        yield i*i   # yield
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
#### 12 python中重载方法
python中没有重载方法，因为python中的方法变量数量可以是任意的，同时也可以指定默认值

#### 13 python变量查找顺序
先从局部——全局的方式
本地作用域（Local）→当前作用域被嵌入的本地作用域（Enclosing locals）→全局/模块作用域（Global）→内置作用域（Built-in）

#### 14 GIL锁
GIL是什么？全局解释锁，python设计之初为了线程安全而做的
在python多线程下，**每个线程的执行方式**：
1. 获取GIL
2. 执行代码知道sleep或者python虚拟机将其挂起
3. 释放GIL
**可见，某个线程想要执行，必须先拿到GIL，我们可以把GIL看作是“通行证”，并且在一个python进程中，GIL只有一个。拿不到通行证的线程，就不允许进入CPU执行**
GIL锁释放逻辑是：在python2.x里，GIL的释放逻辑是当前线程遇见`IO操作`或者`ticks计数达到100(很容易到达)`（ticks可以看作是python自身的一个计数器，专门做用于GIL，每次释放后归零，这个计数可以通过 sys.setcheckinterval 来调整），进行释放

IO型任务 `多线程`  cpu密集型任务 `多进程`

#### 15 __new__和__init__的区别
- `__new__` 平时很少使用(因为继承父类的object默认就有这个方法)  __init__ 经常会使用
- `__new__` 方法的作用是**创建实例**对象, __init__ 的作用**只是初始化**
- `__new__` 静态方法(如果它不是静态方法就无法创建实例)  __init__ 实例方法
实例对象创建过程：先调用`__new__()`，再调用`__init__()`
- 调用实例对象代码xiaoming = Student('xiaoming',175)；
- 传入name和height的参数，执行Student类的__new__()方法，该方法返回一个类的实例，通常会用父类super(Student,cls).__new__(cls)，__new__()产生的实例即__init__()的`self`, 如果在实例方法中，没有`self`字段，就无法被实例调用
- 用实例来调用`__init__()`方法，进行初始化实例对象的操作 **初始化方法是实例调用的**

python中创建顺序：`元类(metaclass)`可以通过方法 `__metaclass__`创造了`类(class)`，而`类(class)`通过方法`__new__`创造了实例`(instance)`

参考链接：https://www.cnblogs.com/jayliu/p/9013155.html

#### 16 单例模式
单例模式是非常常见的设计模式，在python中实现也很简单
1. **import方法导入** 在类模块中先定义个实例对象，后续需要使用的地方 都导入该实例对象
```
# mysingleton.py
class My_Singleton(object):
    def foo(self):
        pass

my_singleton = My_Singleton()  # 类模块中创建实例对象

# to use
from mysingleton import my_singleton  # 后续需要使用的地方 都导入实例对象

my_singleton.foo()
```
2. **new 关键字**
定义一个class变量(用于存储实例变量)
```
class Single(object):
    _instance = None    # 这里定义了一个类变量 共享变量  
    def __new__(cls, *args, **kw):
        if cls._instance is None:
            cls._instance = object.__new__(cls, *args, **kw)   # 创建实例对象
        return cls._instance  
    def __init__(self):
        pass

single1 = Single()
single2 = Single()
print(id(single1) == id(single2))
```
3. **类装饰器实现单例**
这种方式更好，只要想实例单例的`class`,引入该类装饰器就好，节省代码
这里利用了python class的`__call__`属性：它的作用就是表明某个实例对象可以向函数一样通过`()`的方式访问
```
# 思路：定义一个字典，字典的key就是class
class Singleton(object):
    def __init__(self, cls):
        self._cls = cls
        self._instance = {}
    def __call__(self):
        if self._cls not in self._instance:
            self._instance[self._cls] = self._cls()
        return self._instance[self._cls]

@Singleton
class Cls2(object):
    def __init__(self):
        pass

cls1 = Cls2()
cls2 = Cls2()
print(id(cls1) == id(cls2))
```
**__call__ 案例**
```
class Student(object):
    def __init__(self, name):
        self.name = name
    def __call__(self):
        print('My name is %s.' % self.name)
>>> s = Student('Michael')
>>> s() # self参数不要传入
My name is Michael.
```
参考链接：https://zhuanlan.zhihu.com/p/37534850

#### 17 lambda函数
lambda就是`匿名函数`,就是减少代码量
```
lambda x: x * x
等价于 ==>>>
def f(x):
    return x * x
```
关键字`lambda`表示匿名函数，冒号前面的`x`表示函数参数  `省略`掉**函数名、函数()和retrun**

#### 18 高阶函数
1. **map函数**
`map/reduce`来源于Google大名鼎鼎的那边大数据开山之作论文

map()函数接收两个参数，一个是`函数`，一个是`Iterable`，map将传入的函数依次作用到序列的每个元素，并把结果作为新的`Iterator`返回。
```
In [80]: map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9])
Out[80]: [1, 4, 9, 16, 25, 36, 49, 64, 81]

```
2.**filter函数**
filter()函数接收两个参数，一个是`函数`，一个是`Iterable`，map将传入的函数依次作用到序列的每个元素，只是作用的函是用来`过滤`的.
```
def is_odd(n):
    return n % 2 == 1
list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
```
#### 19 python垃圾回收机制
Python GC主要使用引用计数（`reference counting`）来跟踪和回收垃圾。在引用计数的基础上，通`标记-清除`（mark and sweep）解决容器对象可能产生的循环引用问题，通过“分代回收”（generation collection）以空间换时间的方法提高垃圾回收效率.

#### 20 super()方法
super()解决的是多继承，子类调用父类方法的问题
```
在python 3中  
class ChildB(Base):
    def __init__(self):
        super().__init__()    
        
在python 2中
super(ChildB, self).__init__()   
```
#### 21 MRO顺序 
多class继承顺序问题：当子类访问父类方法的时候，如果出现重名的的方法，通过MRO规则决定调用哪个
http://www.srikanthtechnologies.com/blog/python/mro.aspx

#### 22 装饰器wraps的作用
[python装饰器的wraps作用](https://blog.csdn.net/liuskyter/article/details/80357647)

#### 23 双层装饰器
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

