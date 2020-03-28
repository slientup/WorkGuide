- [Junit是什么](#Junit是什么)
- [idea中如何使用Junit](#idea中如何使用Junit)
- [JUnit断言](#JUnit断言)
- [JUnit注解](#JUnit注解)
- [JUnit注解执行顺序图](#JUnit注解执行顺序图)
- [JUnit的一些注意事项](#JUnit的一些注意事项)
- [Junit测试用例](#Junit测试用例)
- [参考链接](#参考链接)

#### Junit是什么
Junit是java中的最小单元的测试，程序中最小单元是方法，单元测试就是验证方法是否符合预期。

#### idea中如何使用Junit
1. 安装junit插件  
2. maven项目中包含junit测试单元的jar  
3. 在需要测试的java类中输入快捷键`ctrl+shift+T` 即创建该类对应的测试类  

#### JUnit断言
断言就是判断方法是否符合预期 如果不符合就报错  常用断言方法：
1. void `assertEquals`(boolean expected, boolean actual):检查两个变量或者等式是否平衡
2. void `assertTrue`(boolean expected, boolean actual):检查条件为真
3. void `assertFalse`(boolean condition):检查条件为假
4. void `assertNotNull`(Object object):检查对象不为空
5. void `assertNull`(Object object):检查对象为空
6. void `assertSame`(boolean condition):assertSame() 方法检查两个相关对象是否指向同一个对象
7. void `assertNotSame`(boolean condition):assertNotSame() 方法检查两个相关对象是否不指向同一个对象
8. void `assertArrayEquals`(expectedArray, resultArray):assertArrayEquals() 方法检查两个数组是否相等
#### JUnit注解
1. `@Test`:这个注释说明依附在 JUnit 的 public void 方法可以作为一个测试案例。
2. `@Before`:有些测试在运行前需要创造几个相似的对象。在 public void 方法加该注释是因为该方法需要在 test 方法前运行。
3. `@After`:如果你将外部资源在 Before 方法中分配，那么你需要在测试运行后释放他们。在 public void 方法加该注释是因为该方法需要在 test 方法后运行。
4. `@BeforeClass`:在 public void 方法加该注释是因为该方法需要在类中所有方法前运行。
5. `@AfterClass`:它将会使方法在所有测试结束后执行。这个可以用来进行清理活动。
6. `@Ignore`:这个注释是用来忽略有关不需要执行的测试的。



#### JUnit 注解执行顺序图
- `beforeClass()`: 方法首先执行，并且只执行一次。
- `afterClass()`:方法最后执行，并且只执行一次。
- `before()`:方法针对每一个测试用例执行，但是是在执行测试用例之前。
- `after()`:方法针对每一个测试用例执行，但是是在执行测试用例之后。
- 在 `before`() 方法和 `after`() 方法之间，执行每一个测试用例。

![注解执行顺序](https://pics5.baidu.com/feed/7dd98d1001e93901196bf37b19bdc3e237d196e4.jpeg?token=77d00b8969dea7e94068b89374fdcdef&s=09A47C32E3C741EB08D5BDDB000010B20)

#### JUnit的一些注意事项：
1. 测试方法必须使用`@Test`修饰
2. 测试方法必须使用`public void`进行修饰，不能带参数
3. 测试代码的包应该和被测试代码包结构保持一致

#### Junit测试用例
测试案例:
![测试用例](https://pics1.baidu.com/feed/d009b3de9c82d158436c798bed5b8eddbd3e42f0.jpeg?token=c2525a1b12b71981ce1660c1ced583d8&s=3281B14CD2BEB66E5AECCC0B0000A0C10)
![测试结果](https://pics1.baidu.com/feed/00e93901213fb80e76792d585780b82bb838946f.jpeg?token=39f79bfb6d385c94f8d690a9a0d13f0a&s=3A84A14CFAE2F7744E45440F0000F0C0)
#### 参考链接
- [使用JUnit进行单元测试](https://www.jianshu.com/p/a3fa5d208c93)
- [junit单元测试详解](https://baijiahao.baidu.com/s?id=1647361249282882030&wfr=spider&for=pc)


