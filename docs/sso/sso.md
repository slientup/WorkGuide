
### apereo cas 安裝

#### 1. 默认搭建过程

- 项目配置目录：`/etc/cas`
- 配置文件：`/etc/cas/config/cas.properties`
- key文件 `/etc/cas/keystore`

搭建过程
1. `git pull cas-overlay-template`
   - `https://github.com/apereo/cas-overlay-template`
2. build war  
   - `gradlew.bat clean build` 编译成`war包`
3. 运行war
   - `java -jar cas.war`  不需要单独安装tomcat
4. 运行过程中会提示没有`/etc/cas/keystore`文件不存在 这个目录等价于windows的`C:\etc\cas\keystore` 所以将文件复制到`C:\etc\cas\`目录下面就可以

5. 模板中的配置目录`/etc/cas`,记得要将里面的内容都复制到`C:\etc\cas\`下面，其中`/etc/cas/config/cas.properties` 是配置文件

6. 使用默认用户名和密码就可以登录了`casuser/Mellon`


参考文档 
- `https://www.selinux.tech/architecture/cas/cas-install`
- 官方文档 https://github.com/apereo/cas-overlay-template

#### 2.docker环境搭建
> 上面这种还需要自己搭环境，那有没有更简单的方法，其实过程是非常麻烦的。有没有类似docker，一条命令就起来。


参考文档：[基于docker搭建CAS](https://www.jianshu.com/p/475fe96031d4)

1. 生成密钥  thekeystore

    `keytool -genkeypair -alias cas -keyalg RSA -keypass changeit -storepass changeit -keystore ./thekeystore   -dname "CN=cas.example.org,OU=Example,OU=Org,C=AU" -ext SAN="dns:example.org,dns:localhost,ip:127.0.0.1"`

2. 启动容器（直接挂载了key 这样就不会报错）

    `docker run  --name cas -p 8443:8443 -p 8080:8080 -v $home/thekeystore:/etc/cas/thekeystore apereo/cas:6.3.4 /bin/sh /cas-overlay/bin/run-cas.sh`

3. 配置文件`/etc/cas/config/cas.properties` 直接将这个文件docker cp到本地 修改后再挂载


4. 重新拉起容器

    `docker run  --name cas -p 8443:8443 -p 8080:8080 -v $home/thekeystore:/etc/cas/thekeystore -v $home/cas.properties:/etc/cas/config/cas.properties apereo/cas:6.4.0-RC4 /bin/sh /cas-overlay/bin/run-cas.sh`

5. 修改配置文件

- 增加一个本地认证用户
`cas.authn.accept.users=zxzeng::zxzeng` 重启容器  测试用该账号登录成功

- 增加ldap认证 （很遗憾在增加ldap的时候 docker容器无法启动ldap，应该是容器本身没有包含） 这里应该改进，即默认将所有常用的auth类型依赖都添加，当用户需要使用哪种的时候，就添加哪种，这是最好的，避免自己搭建开发环境。


#### 3.ldap认证配置
> 采用自己打包的方式，并没有使用docker 因为docker存在异常

参考文档：[CAS 集成LDAP](https://www.selinux.tech/architecture/cas/cas-ldap)

两个步骤
1. 加载依赖包，然后重新打包war
2. 在`cas.properties`中配置ldap的信息
    ```
    ldap-url=ldap://xxxxx:389       // url
    ldap-dnformat=cn=%s,cn=users,dc=test,dc=com   // 这个是用来填充用户的格式 比如ad中用户的格式是  cn=zxzeng,cn=users,dc=baidu,dc=com
    ldap-base-dn=CN=Users,DC=test,DC=com
    ldap-bind-dn=CN=xxxxx,OU=SVC,OU=xxxx,DC=test,DC=com
    ldap-bind-credential= xxx  // 密码

    cas.authn.ldap[0].password-policy.groovy.location=
    cas.authn.ldap[0].principal-transformation.groovy.location=
    cas.authn.ldap[0].base-dn=${ldap-base-dn}
    cas.authn.ldap[0].bind-dn=${ldap-bind-dn}
    cas.authn.ldap[0].bind-credential=${ldap-bind-credential}
    cas.authn.ldap[0].dn-format=${ldap-dnformat}
    cas.authn.ldap[0].ldap-url=${ldap-url}
    cas.authn.ldap[0].search-filter=(cn={user})  // 这里是搜索用户的 看具体的格式
    cas.authn.ldap[0].type=DIRECT
    cas.authn.ldap[0].password-encoder.encoding-algorithm=
    cas.authn.ldap[0].password-encoder.type=NONE
    ```
3. 输入AD域控账号登录成功

上面第二步是最难的，这里需要你要了解自己公司的AD域结构，然后通过debug调试信息，慢慢比对才找到最终对的格式，而且cas 不同版本配置也不一样，这是很难受的


#### 4. cas client 对接
**测试基本信息**
- django框架
- django-cas-ng模块做客户端

**问题一 未认证授权的服务 不允许使用CAS来认证您访问的目标应用**
- **描述：** 通过`python manager runserver 127.0.0.1:8080` 拉起服务，访问api时候跳转到`cas 页面`，但提示`未认证授权的服务 不允许使用CAS来认证您访问的目标应用。`

![](https://files.mdnice.com/user/4251/f25b8335-f591-4e0e-9fe6-91c0243db6c4.png)

- **解决思路：** 默认情况下cas只放开了https，对http服务并没有放开，所以需要配置放开`http`，这需要用到`cas-management`,哎安装这个过程又是一件痛苦的事情，服务就从来没有拉起来过，我选择了将测试django 以https的方式启动

**问题二： `cas login` 认证能通过，但是通过之后不断重复重定向 **
- **描述** `python manage.py runserver_plus --cert cert 127.0.0.1:8000`  以这条命令启动，能正常弹出`cas login` 认证也能通过，但是通过之后不断重复重定向。
- **解决思路**： 通过查看源码发现，不断重复是因为没有解析到用户，我就非常疑惑，为什么没有用户，看cas的debug信息都没什么异常信息，我被卡住了，没有思路，卡了好一会儿，我就打算开始调试`django-cas-ng`的源码 一步一步测试的过程发现它调用了`backend`，我隐隐觉得是我配置没有写全，配置是我从我以前另一个项目拿过来的，上次都一年前了，我去看了官网配置，果然是在settings中配置不全导致的
```
MIDDLEWARE_CLASSES = (
    'django.middleware.common.CommonMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django_cas_ng.middleware.CASMiddleware',
    ...
)

AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'django_cas_ng.backends.CASBackend',
)
```

哎 重新配置后解决问题

**问题三：cas认证通过后返回client 多个属性 比如`email` `mobile`等 **

1. `cas.properties` 增加如下配置：
    ```
    # ldap 返回多个属性配置
    cas.authn.attribute-repository.ldap[0].base-dn=${ldap-base-dn}
    cas.authn.attribute-repository.ldap[0].bind-dn=${ldap-bind-dn}
    cas.authn.attribute-repository.ldap[0].ldap-url=${ldap-url}
    cas.authn.attribute-repository.ldap[0].search-filter=(cn={user})
    cas.authn.attribute-repository.ldap[0].bind-credential=${ldap-bind-credential}
    # 具体的属性键值对 如果需要添加则再增加
    cas.authn.attribute-repository.ldap[0].attributes.givenName=first_name
    cas.authn.attribute-repository.ldap[0].attributes.sn=last_name
    cas.authn.attribute-repository.ldap[0].attributes.mail=email
    cas.authn.attribute-repository.ldap[0].attributes.mobile=mobile
    ```
    参考连接：[LDAP Attribute Resolution](https://apereo.github.io/cas/development/integration/Attribute-Resolution-LDAP.html)
2. 做了上述配置之后，通过cas login页面看返回的属性中增加了上述的中的属性，但通过`django-cas-ng`的client却没有相应的属性

3. 我需要判断是`django-cas-ng`的问题还是`cas server`的问题,`django-cas-ng`中打印返回的信息
    ```
            username, attributes, pgtiou = client.verify_ticket(ticket)
            print("auth",username,attributes,pgtiou)
    ```
    返回结果如下：（**说明已经返回属性了**）
      ```
      auth zhixiong.zeng {'credentialType': 'UsernamePasswordCredential', 'firstName': 'Zhixiong', 'lastName': 'Zeng', 'isFromNewLogin': 'true', 'authenticati
      onDate': '2021-06-16T08:31:00.187228Z', 'authenticationMethod': 'LdapAuthenticationHandler', 'successfulAuthenticationHandlers': 'LdapAuthenticationHand
      ler', 'mobile': 'xxxxx', 'longTermAuthenticationRequestTokenUsed': 'false', 'email': 'zhixiong.zeng@123.com'} None
      ````
4. 继续查看源码，发现默认只添加用户名，但可以通过设置`CAS_APPLY_ATTRIBUTES_TO_USER`字段实现其他字段的添加，`CAS_RENAME_ATTRIBUTES`该字段重命名属性，增加如下配置
    ```
    CAS_APPLY_ATTRIBUTES_TO_USER = True
    CAS_RENAME_ATTRIBUTES = {
        "firstName": "first_name",
        "lastName": "last_name",
    }
    ```


#### cas-management安装
> 无论docker 还是overlay 打包的方式都没有成功，但记录下

- docker方式
 1. `docker run -d -p 8081:8080 -p 8444:8443 --name="cas-management" apereo/cas-management:6.3.1`
 2.   `docker run -d -p 8081:8080 -p 8444:8443 --name="cas-management" -v $home/thekeystore:/etc/cas/thekeystore -v $home/management.properties:/etc/cas/config/management.properties apereo/cas-management:6.3.1`


- 参考文档 [CAS Service Management](https://www.selinux.tech/architecture/cas/cas-management-install)

#### 5. 关于允许http协议的问题
> 之前我通过搜索引擎查到，需要使用`cas-management`来实现，今天在阅读文档的时候，发现可以不使用该方案，直接在`cas-server`服务器中安装依赖包就可以,应该是我的理解问题，**实际上本身就需要在`cas-server`中安装依赖包,做相应的配置，而`cas-management`只是外挂系统，对这些服务做一个管理，用户注册的过程不需要经过`cas-management`**

1. 添加依赖，并重新打包
    ```
    implementation "org.apereo.cas:cas-server-support-json-service-registry:${project.'cas.version'}"
    ```
2. 在`cas.properties`中做配置
    ```
    cas.service-registry.core.init-from-json=true
    cas.serviceRegistry.json.location=file:/etc/cas/services
    ```
3. 在`/etc/cas/services`文件配置服务注册允许规则
    ```
    {
      "@class" : "org.apereo.cas.services.RegexRegisteredService",
      "serviceId" : "^(https|imaps|http)://.*",
      "name" : "web",
      "id" : 10000001,
      "evaluationOrder" : 10
    }

    ```
参考链接：
- [JSON Service Registry](https://apereo.github.io/cas/development/services/JSON-Service-Management.html)
- [CAS Service Management](https://www.selinux.tech/architecture/cas/cas-management-install)


#### 完整配置

[完整配置](https://github.com/slientup/cas-overlay-template)

#### 重定义login页面

[CAS单点登录-自定义登录页、修改编辑登录页](https://www.cnblogs.com/hooly/p/12784397.html)

#### 故障指南


[故障指南](https://apereo.github.io/cas/development/installation/Troubleshooting-Guide.html)
