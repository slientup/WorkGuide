### 常用设计模式
设计模式分三类：创建类模式、创建型模式和结构型模式  
#### 创建类模式
1. 单例模式  始终只有一个实例

      public class Singleton {

          //volatile保证，当uniqueInstance变量被初始化成Singleton实例时，多个线程可以正确处理uniqueInstance变量
          private volatile static Singleton uniqueInstance;
          private Singleton() {
          }
          public static Singleton getInstance() {
             //检查实例，如果不存在，就进入同步代码块
              if (uniqueInstance == null) {
                  //只有第一次才彻底执行这里的代码
                  synchronized(Singleton.class) {
                     //进入同步代码块后，再检查一次，如果仍是null，才创建实例
                      if (uniqueInstance == null) {
                          uniqueInstance = new Singleton();
                      }
                  }
              }
              return uniqueInstance;
          }
      }
2. 代理模式  
代理模式中间人，比如法律上面的代理律师一样，最终用户不能直接访问我，通过我的代理访问我
```
    总接口
     public interface ZiRanRen {
         void Quanli();
     }
    委托人：
    public class MaYun implements ZiRanRen {
        public void eat() {
            System.out.println("今天吃满汉全席");
        }
        public void drink() {
            System.out.println("今天喝大西洋");
        }
        @Override
        public void Quanli() {
            System.out.println("我赋予我的代理律师来行使这些权利,此时代理律师全权代理我处理某些事务");
        }
    }
    public class LvShi implements ZiRanRen {
        @Override
        public void Quanli() {
            new MaYun().Quanli();   //这里的new MaYun()是核心
        }
    }
    客户端测试程序：
    public class Clienter {
        public static void main(String[] args) {
            ZiRanRen ls = new LvShi();
            ls.Quanli();
        }
    }
```
3. 观察者模式
`发布的订阅`的模式,`通知模型`的应用，设计模式中分 `subject`和`Observer`角色,`subject`中需要存放`observer`的信息list，然后再写一个通知方法



