- 问题1：vue前端axios提交数据携带cookie信息 用于cas认证
- 问题2：vue前端axios提交数据 django post 403报错
- 问题3： vue前端通过axios提交时间类型的参数时时区自动转换问题。
- 问题4： vue定时更新setInterval整个页面抖动
- 问题5：element-ui字体图标样式部署到服务器后失效
- 问题6: vue table表中时间列格式化的问题
- 问题7 iframe height="100%"不生效的问题
- 问题8 element ui 自定义图标和路径
- 问题9 vue 通过设置环境变量，多环境部署
- 问题10 vue router 传参
##### 问题1：vue前端axios提交数据携带cookie信息 用于cas认证
cas认证本质是cookie信息

解决思路：
1. 在axios配置中添加`withCredentials: true`
2. 在django后端`setting.py`中添加`CORS_ALLOW_CREDENTIALS = True`

在本地电脑环境做测试的时候，在上述两个步骤都做了之后依然未携带`cookie`信息，前端地址用的是`10.33.128.3:8090`(我的pc实际地址),后端地址用的是`127.0.0.1:9090`

解决措施：将前端的地址更改为`127.0.0.1:8090` 就可以携带cookie信息


参考链接：[axios 跨域处理以及带 cookies 的请求](https://www.jianshu.com/p/161cd33af3a0)


##### 问题2：vue前端axios提交数据 django post 403报错

解决方法1 post方法的时候在头部带上`csrftoken`信息
```
  if(config.method == 'post'){
    config.headers['X-CSRFToken'] =Cookies.get('csrftoken');
  }
  return config
```
参考链接：[django+vue axios的post请求403 csrftoken问题](https://www.jianshu.com/p/c7c491fd67ee)

解决方法2：直接关闭`csrftoken`认证 通过中间件的方式 一劳永逸

参考链接：[django 解决使用django rest framework时web请求报CSRF Failed: CSRF cookie not set](http://www.chenxm.cc/article/589.html)


##### 问题3： vue前端通过axios提交时间类型的参数时时区自动转换问题。

通过 Chrome 浏览器的Inspect功能，查看Network，发现时间参数被自动修改了，我们选定的时间是`2019-07-12 00:00:00`，在传输的时候却被修改为`2019-07-11 16:00:00`


使用`moment`组件
```
	const moment = require('moment')

    const startDateStr = moment(startDate).format('YYYY-MM-DD HH:mm:ss')
    const endDateStr = moment(endDate).format('YYYY-MM-DD HH:mm:ss')
```

参考链接：[解决 axios 提交时间类型参数遇到的时区自动转换问题](https://cloud.tencent.com/developer/article/1518582)


##### 问题4： vue定时更新setInterval整个页面抖动

如果直接粗暴对整个表进行定时刷新就会出现这种情况，只需要对会变化的字段进行刷新就会不会出现页面抖动


会出现页面抖动

```
      getList() {
        fetchList().then(response => {
            this.TaskList=JSON.parse(response.data);
        });
      },
```

不会出现抖动
```
       getLogList(){
            fetchList().then(response => {
                this.TaskList[0].upgradeTable[0].log=this.TaskList[0].upgradeTable[0].log+"1234343"+"\n"
            });
        },
```


##### 问题5：element-ui字体图标样式部署到服务器后失效

参考：https://blog.csdn.net/youyudexiaowangzi/article/details/106901458

##### 问题6: vue table表中时间列格式化的问题
参考连接：https://blog.csdn.net/x_lord/article/details/70225481
1. 通过在需要格式化的列中指定格式器`:formatter="dateFormat"`
```
 <el-table-column prop="created_time" label="创建时间" :formatter="dateFormat" sortable/>
```
2. 定义formatter
 ```
 methods: {

    dateFormat: function (row, column) {
      const date = row[column.property]
      return moment(date).format('YYYY-MM-DD HH:mm:ss')
    },
```

##### 问题7 iframe height="100%"不生效的问题
参考连接：https://www.jianshu.com/p/ae8f417e0824

有两种方案，最终选择了第二种方案，大小自适应。(第一种怎么试都不行)
```
<template>
  <iframe id="main" width="100%" :height="height" frameborder="0" src="xxx" />
</template>
<script>
export default {
  name: 'Dashboard',
  data() {
    return {
      height: null
    }
  },
  created() {
    this.height = document.documentElement.clientHeight + 'px'
  }
}
</script>
```
##### 问题8 element ui 自定义图标和路径
参考链接：https://segmentfault.com/a/1190000025176437

1. 定义  注意这里的图标引入路径
```
.el-icon-custom-grafana{
  background: url('../../icons/svg/grafana_logo.png') center no-repeat;
  background-size: cover;
}
.el-icon-custom-grafana:before{
  content: "替";
  font-size: 18px;
  visibility: hidden;
}
```
2. 引用
`<el-button  icon="el-icon-custom-grafana"  size="mini"`
 
 
##### 问题9 vue 通过设置环境变量，多环境部署
参考连接：
- https://github.com/Charles2mx/vue-admin-template-1   

未使用vue cli的多环境配置文件(.env.development等)因为这种方式修改url，就必须重新打包镜像，这在实际k8s场景不合适

1. 第一步在`public`文件夹下面创建一个`config.js`,一定要这个文件夹，因为这个文件夹里面的文件`npm run build`会保留
```
window.config= {
  BACKEND_BASE_API : "http://localhost:8080/",
  WEB_LOGIN_URL: "http://localhost:9528/#/login",
}
```
2. 在index.html文件中
```
    <link rel="icon" href="<%= BASE_URL %>test.jpg">
    <script type="text/javascript" src="<%= BASE_URL %>config.js"></script>    // 引入这一行
```

3. 引入变量
```
const service = axios.create({
  baseURL: window.config.BACKEND_BASE_API, // url = base url + request url
  timeout: 5000, // request timeout
  withCredentials: true
})
```
4. 只需要在不同的环境重写`config.js`就可以

5. 文件目录：
   public
       - config.js
       - test.jpg
       - index.html


##### 问题10 vue router 传参
参考连接：https://segmentfault.com/a/1190000012393587  方案三
1. 设置this.$router.push
```
    this.$router.push({
          path: '/describe',
          query: {
            id: id
          }
        })
```
2. 对应路由配置
```
   {
     path: '/describe',
     name: 'Describe',
     component: Describe
   }
```
3. Describe组件获取参数
`this.$route.query.id`
      

