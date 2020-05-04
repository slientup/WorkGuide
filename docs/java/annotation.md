# java注解
[java中自定义注解](https://www.cnblogs.com/Lyn4ever/p/11594533.html)
[java注解视频](https://www.bilibili.com/video/BV1Vt411g7RP?from=search&seid=4689814467860577055)

当我们在某个类中引入注解的时候，它与当前类的关系是什么啦？它是如何起到相关的作用啦？  
当前类与注解之间的关系，是通过**反射**来关联起来的，只是这些繁琐的代码被对应的框架来实现了，如spring框架

#### 注解本质
```
#MyAnno.java
public @interface MyAnno{
   int age();  //定义age属性
   String name(); //定义name属性
}

javac MyAnno.java 对其进行编译

javap MyAnno.class 对其进行反编译结果如下
public interface MyAnno extends java.lang.annotation.Annotation {

}
```
* 注解格式：
  public @interface 名称{
}
* 注解本质：是一个**接口**，该接口默认继承Annotation接口
* 属性：接口可以定义成员方法，接口可以定义什么就能定义

#### 注解赋值
```
1.注解使用时，给属性赋值
@MyAnno(age = 12, name ="zhangshan")

2.如果只有一个属性需要赋值，并且属性的名称是value，则value可以省略，直接定义值即可

public @interface MyAnno{
   int value();  //定义value
}
@MyAnno(12)

3. 数组赋值时，值使用{}包裹。如果数组中只有一个值，则{}可以省略
```
#### 元注解
```
* 元注解：用于描述注解的注解
		* @Target：描述注解能够作用的位置
			* ElementType取值：
				* TYPE：可以作用于类上
				* METHOD：可以作用于方法上
				* FIELD：可以作用于成员变量上
		* @Retention：描述注解被保留的阶段
			* @Retention(RetentionPolicy.RUNTIME)：当前被描述的注解，会保留到class字节码文件中，并被JVM读取到
		* @Documented：描述注解是否被抽取到api文档中
		* @Inherited：描述注解是否被子类继承

    package cn.itcast.annotation;
    
    import java.lang.annotation.*;
    
    /**
    
     元注解：用于描述注解的注解
         * @Target：描述注解能够作用的位置
         * @Retention：描述注解被保留的阶段
         * @Documented：描述注解是否被抽取到api文档中
         * @Inherited：描述注解是否被子类继承
    
    
     *
     */
    
    @Target({ElementType.TYPE,ElementType.METHOD,ElementType.FIELD})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    public @interface MyAnno3 {
    }
```
#### 自定义程序中使用注解

@注解是如何跟当前`class或者method或者var`绑定在一起的啦？当前类必须通过反射的方式先获取到对应的注解对象，再获取对应的值赋给当前作用目标

```
* 在程序使用(解析)注解：获取注解中定义的属性值
		1. 获取注解定义的位置的对象  （Class，Method,Field）
		2. 获取指定的注解
			* getAnnotation(Class)
			//其实就是在内存中生成了一个该注解接口的子类实现对象

		            public class ProImpl implements Pro{
		                public String className(){
		                    return "cn.itcast.annotation.Demo1";
		                }
		                public String methodName(){
		                    return "show";
		                }
		            }
		3. 调用注解中的抽象方法获取配置的属性值

    package cn.itcast.annotation;
    
    import java.io.InputStream;
    import java.lang.reflect.Method;
    import java.util.Properties;
    
    /**
     * 框架类
     */
    
    
    @Pro(className = "cn.itcast.annotation.Demo1",methodName = "show")
    public class ReflectTest {
        public static void main(String[] args) throws Exception {
    
            /*
                前提：不能改变该类的任何代码。可以创建任意类的对象，可以执行任意方法
             */
    
            //1.解析注解
            //1.1获取该类的字节码文件对象
            Class<ReflectTest> reflectTestClass = ReflectTest.class;
            //2.获取上边的注解对象
            //其实就是在内存中生成了一个该注解接口的子类实现对象
            /*
    
                public class ProImpl implements Pro{
                    public String className(){
                        return "cn.itcast.annotation.Demo1";
                    }
                    public String methodName(){
                        return "show";
                    }
    
                }
     */     //获取当前类的注解类对象
            Pro an = reflectTestClass.getAnnotation(Pro.class);
            //3.调用注解对象中定义的抽象方法，获取返回值
            String className = an.className();
            String methodName = an.methodName();
            System.out.println(className);
            System.out.println(methodName);
    
            //3.加载该类进内存
            Class cls = Class.forName(className);
            //4.创建对象
            Object obj = cls.newInstance();
            //5.获取方法对象
            Method method = cls.getMethod(methodName);
            //6.执行方法
            method.invoke(obj);
        }
    }





