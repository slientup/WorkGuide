### vue-admin-web 学习笔记

### 基础框架
基础框架，是**任何项目**都会使用到的架构代码，包括`结构`和`aop`的抽象
#### 整体流程

**非常非常重要**   路由控制着整个流程，也控制着整个页面的组件渲染，所有页面都父组件`Layout`(页面布局)，然后包含子组件
```js
export const constantRouterMap = [
  {path: '/login', component: () => import('@/views/login/index'), hidden: true},
  {path: '/404', component: () => import('@/views/404'), hidden: true},
  {
    path: '',
    component: Layout,   // 这里非常关键 这是所有的父组件  页面的整体框架
    redirect: '/home',
    children: [{
      path: 'home',
      name: 'home',
      component: () => import('@/views/home/index'),
      meta: {title: '首页', icon: 'home'}
    }]
  }
]
```
`home`页面就是通过父组件`Layout`和子组件`/views/home/index`构成  这里解决了动态路由的生成问题
**实现思路：**  存储一个共享变量 `layout`调用这个共享变量 加载每个页面都包含`layout`
1. 首次访问页面时，获取用户的访问页面访问权限,然后保存到`store`共享变量`routers`中  
```
state: {
    routers: constantRouterMap,   // 存储路由信息 默认是常量
    addRouters: []   //存储动态的路由信息
  }
```
2. 其次每个页面默认加载的时候都会渲染`Layout`组件，这个组件在加载的时候默认会加载'routers'字段

```js
export const asyncRouterMap = [
  {
    path: '/pms',
    component: Layout,     // 这个Layout组件非常重要
    redirect: '/pms/product',
    name: 'pms',
    meta: {title: '商品', icon: 'product'},
    children: [{
      path: 'product',
      name: 'product',
      component: () => import('@/views/pms/product/index'),
      meta: {title: '商品列表', icon: 'product-list'}
    },
      {
        path: 'addProduct',
        name: 'addProduct',
        component: () => import('@/views/pms/product/add'),
        meta: {title: '添加商品', icon: 'product-add'}
      },   
```

#### 权限验证(是否登录)
这段代码存放在`permission.js` 通过`router.beforeEach`来判断用户登录,这段代码运行在路由`router`之前，通过`router.afterEach`结束进度条,运行在路由之后
```js
import router from './router'
import store from './store'
import NProgress from 'nprogress' // Progress 进度条
import 'nprogress/nprogress.css'// Progress 进度条样式
import { Message } from 'element-ui'
import { getToken } from '@/utils/auth' // 验权

const whiteList = ['/login'] // 不重定向白名单
router.beforeEach((to, from, next) => {
  NProgress.start()
  if (getToken()) {
    if (to.path === '/login') {
      next({ path: '/' })
      NProgress.done() // if current page is dashboard will not trigger	afterEach hook, so manually handle it
    } else {
      if (store.getters.roles.length === 0) {
        store.dispatch('GetInfo').then(res => { // 拉取用户信息
          let menus=res.data.menus;
          let username=res.data.username;
          store.dispatch('GenerateRoutes', { menus,username }).then(() => { // 生成可访问的路由表
            router.addRoutes(store.getters.addRouters); // 动态添加可访问路由表
            next({ ...to, replace: true })
          })
        }).catch((err) => {
          store.dispatch('FedLogOut').then(() => {
            Message.error(err || 'Verification failed, please login again')
            next({ path: '/' })
          })
        })
      } else {
        next()
      }
    }
  } else {
    if (whiteList.indexOf(to.path) !== -1) {
      next()
    } else {
      next('/login')
      NProgress.done()
    }
  }
})

router.afterEach(() => {
  NProgress.done() // 结束Progress
})

```
#### 对axios请求进行封装
请求封装代码存放在`src/utils/requests`中,会使用到拦截器，即对请求和响应做一些处理
```js
import axios from 'axios'
import { Message, MessageBox } from 'element-ui'
import store from '../store'
import { getToken } from '@/utils/auth'

// 创建axios实例
const service = axios.create({
  baseURL: process.env.BASE_API, // api的base_url
  timeout: 15000 // 请求超时时间
})

// request拦截器
service.interceptors.request.use(config => {
  if (store.getters.token) {
    config.headers['Authorization'] = getToken() // 让每个请求携带自定义token 请根据实际情况自行修改
  }
  return config
}, error => {
  // Do something with request error
  console.log(error) // for debug
  Promise.reject(error)
})

// respone拦截器
service.interceptors.response.use(
  response => {
  /**
  * code为非200是抛错 可结合自己业务进行修改
  */
    const res = response.data
    if (res.code !== 200) {
      Message({
        message: res.message,
        type: 'error',
        duration: 3 * 1000
      })

      // 401:未登录;
      if (res.code === 401) {
        MessageBox.confirm('你已被登出，可以取消继续留在该页面，或者重新登录', '确定登出', {
          confirmButtonText: '重新登录',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          store.dispatch('FedLogOut').then(() => {
            location.reload()// 为了重新实例化vue-router对象 避免bug
          })
        })
      }
      return Promise.reject('error')
    } else {
      return response.data
    }
  },
  error => {
    console.log('err' + error)// for debug
    Message({
      message: error.message,
      type: 'error',
      duration: 3 * 1000
    })
    return Promise.reject(error)
  }
)

export default service
```

### vue路由动态权限认证
这一步用来实现动态路由的加载  通过`addRoutes`来实现
![vue路由动态权限认证](https://jack-cool.github.io/2019/08/04/vue%E8%B7%AF%E7%94%B1%E5%8A%A8%E6%80%81%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/)

### es6常用语法
1. const{} 实现右侧属性的对等赋值
        const { menus } = data;  等价于将`data.menus`赋值给`menus`
        const { username } = data;
2.数组的filter方法 过滤元素
```js
const accessedRouters = asyncRouterMap.filter(v => {    // es6中的数组过滤
          //admin帐号直接返回所有菜单
          // if(username==='admin') return true;
          if (hasPermission(menus, v)) {
```
3.store中action方法必须要有`{commit}`,通过该方法调用`mutation`
```js
    GenerateRoutes({ commit }, data) {
```
4.定时获取数据`setTimeout`

5.`$route.matched` 一个数组，包含当前路由的所有**嵌套路径**片段的路由记录

6. `$refs`通过这个对象访问`dom`元素

7. 传递路由参数信息和表单对象信息`$route`有哪些字段 **这个全局是共享的** 可以用vue谷歌插件来看 非常智能  
```js
 updateProductCate(this.$route.query.id, this.productCate).then(response => {  // 传递对象信息和id信息
```
8.前端组成展示
 ```this.$router.push({path:'/pms/updateProductCate',query:{id:row.id}});``` 
 url的样式：`http://localhost:8090/#/pms/updateProductCate?id=1`  这里会携带`id`信息

9.编辑按钮没有使用单独的模态框的模式，而是直接给了一个页面 默认情况下页面是hidden 这还是一个新的思路 平时习惯模态框
```   这个:is-edit 是props属性
父组件的配置
<template>
  <product-attr-detail :is-edit='true'></product-attr-detail>  //通过在调用子组件的时候传递该属性
</template>
子组件的配置  子组件接收props属性  
    props: {    // 代表里面的值可以通过父组件传递过来
      isEdit: {
        type: Boolean,
        default: false  这是该属性默认的值
      }
    }
含义是父组件向子组件传递给`isEdit`=true的值
```

10. data数据指定默认的值，同时还可以随时覆盖
`productCate: Object.assign({}, defaultProductCate)`
Object.assign()方法用于将所有可枚举的属性的值从一个或多个源对象复制到目标对象，它将返回目标对象

11.字典list操作
```js
            for (let i = 0; i < response.data.length; i++) {
              this.filterProductAttrList.push({
                key: Date.now() + i,
                value: [response.data[i].attributeCategoryId, response.data[i].attributeId]
              })
            }
          }
```











 
 
 
 
 



