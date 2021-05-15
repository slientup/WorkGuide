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

