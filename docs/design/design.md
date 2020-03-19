- [单例模式](#单例模式)
- [单例模式](#代理模式)
- [观察者模式](#观察者模式)
- [JDK事件](#JDK事件)

### 常用设计模式
设计模式分三类：创建类模式、创建型模式和结构型模式  
#### 创建类模式
##### 单例模式      
始终只有一个实例

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
##### 代理模式  
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

##### 观察者模式

`发布的订阅`的模式,`通知模型`的应用，设计模式中分 `subject`和`Observer`角色,`subject`中需要存放`observer`的信息list，然后再写一个通知方法  
可参考： https://juejin.im/post/5c712ab56fb9a049a7127114#heading-4

##### JDK事件
观察者模式的应用：这段代码从UserService和EventApp开始看 https://juejin.im/post/5dd284d1e51d45401f0781ef

      public class User {
          private String username;
          private String password;
          private String sms;
          public User(String username, String password, String sms) {
              this.username = username;
              this.password = password;
              this.sms = sms;
          }  
      }
      public interface UserListener extends EventListener {
          void onRegister(UserEvent event);
      }
      public class SendSmsListener implements UserListener {
          @Override
          public void onRegister(UserEvent event) {
              if (event instanceof SendSmsEvent) {
                  Object source = event.getSource();
                  User user = (User) source;
                  System.out.println("send sms to " + user.getUsername());
              }
          }
      }
      public class UserEvent extends EventObject {
          public UserEvent(Object source){
              super(source);
          }
      }
      public class SendSmsEvent extends UserEvent {
          public SendSmsEvent(Object source) {
              super(source);
          }
      }
      public class UserService {
          private List<UserListener> listenerList = new ArrayList<>();
          //当用户注册的时候，触发发送短信事件
          public void register(User user){
              System.out.println("name= " + user.getUsername() + " ,password= " + 
                                            user.getPassword() + " ,注册成功");
              publishEvent(new SendSmsEvent(user));
          }
          public void publishEvent(UserEvent event){
              for(UserListener listener : listenerList){
                  listener.onRegister(event);
              }
          }
          public void addListeners(UserListener listener){
              this.listenerList.add(listener);
          }
      }
      public class EventApp {
          public static void main(String[] args) {
              UserService service = new UserService();
              service.addListeners(new SendSmsListener());
              //添加其他监听器 ...
              User user = new User("foo", "123456", "注册成功啦！！");
              service.register(user);
          }
      }


