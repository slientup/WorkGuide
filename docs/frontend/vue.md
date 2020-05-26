##  vue现在最流行的前端框架 
> vue是前端框架，是目前最接近后端编程语言思路的框架，对后端程序员来说上手很容易

- [vue基础](#vue基础)
- [vue组件](#vue组件)
- [vuex](#vuex)
- [vue-router](#vue-router)
- [vue项目启动流程](#vue项目启动流程)
- [axios](#axios)
- [js-cookie](#js-cookie)
- [path-to-regexp](#path-to-regexp)
- [nprogress](#nprogress)
- [npm](#npm)
- [vue常用插件](#vue常用插件)



- [参考资料](#参考资料)

## vue基础
> vue属性是vue最重要的特性之一，其中**数据**和**方法**属性最重要

**VUE实例**

### data数据属性
> 该属性等价于`java class`中的属性，值的类型为字典，data 对象中的所有的 `property`加入到 Vue 的**响应式系统**中,**只有data**中的`property`才会与DOM实现动态关联
```javascript
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Reversed message: "{{ reversedMessage() }}"</p>
</div>
//新建一个Vue对象
var vm = new Vue({   
  el: '#example',   //el属性 作用的html对象的范围  在example标签下面
  data: {           //data属性  定义变量
    message: 'Hello'    
  },
methods: {       //方法属性  里面定义方法模块
   reversedMessage: function () {         // reversedMessage 方法名
       return this.message.split('').reverse().join('')
  }
}
```
**注意**：只有当实例被创建时就已经存在于 `data` 中的`property`才是响应式的,实例后创建的属性如`vm.b = 'hi'`不是响应式的

### method方法属性
参考上面举例的`reversedMessage()`就是vue中方法属性的定义方式

### computed计算属性
该属性的特点，就是调用该属性不需要使用方法的`()`方括号，这与python中的`@property`非常类似，计算属性实现的效果方法属性也能实现，为什么要**使用它？**  因为**计算属性**是基于它们的响应式依赖**进行缓存的**,只在相关响应式依赖发生改变时它们才会重新求值，提升性能。
```javascript
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>   ### 这里渲染时候就会调用`reversedMessage`
</div>
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {   reversedMessage 就是计算属性 
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```
### watch监听属性
> 顾名思义，监听某个值发生了变化就触发某些内容 当`question`值发生改变调用对应的方法处理
```javascript
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {    这里question就是监听的对象  
      this.answer = 'Waiting for you to stop typing...'
      this.debouncedGetAnswer()
    }
  }
```
### filters过滤器属性
该属性主要用于在数据展示的时候做一定的处理，对内容进行格式化输出，本质上是一个方法  message **| toupper{过滤器名}** 
```javascript
<div id="myApp">
    <p>{{message}}</p>
    <p>{{message | toupper }}</p>   //过滤器使用的方式  | 过滤器名
    <hr>
    <p>现在的vue.js学习进度是{{num}}({{num | topercentage}})。</p>
</div>
<script>
    var myApp = new Vue({
        el: '#myApp',
        data: {
            message: 'hello vue.js world.',
            num: 0.3
        },
        filters: {
            toupper: function(value){   //过滤器的名字
                return value.toUpperCase();
            },
            topercentage: function(value){   
                return value * 100 + '%';
            },
        },
    });
</script>
```
### 钩子方法
每个 Vue 实例在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等
钩子方法的作用就是在实例被创建前或者后**自动执行**   
常用钩子方法如下：`created`、`mounted`、`updated`和`destroyed` 其中`mounted`的方法通常初始化.


### 常用指令
> 这部分主要是html中的指令
- `v-on` 绑定事件  如v-on:click 单击事件
- `v-model` 绑定vue实例中data属性 在表单元素上做双向绑定 **只能用于form表单框** 
- `v-if`  `v-show` `v-for`

### 事件
> 鼠标点击 移动等触发某个事件   this指当前对象(vue 实例)，event是原生的DOM事件
```Javascript
<div id="myApp">
    <h1>事件处理器</h1>
    <input id="txtName" v-on:keyup="txtKeyup($event)">
    <button id="btnOK" v-on:click="btnClick($event)">OK</button>
</div>
<script>
    var myApp = new Vue({
        el: '#myApp', 
        data: {},
        methods: {
            txtKeyup: function(event){
                this.debugLog(event); this值当前对象  vue实例  event是原生的DOM事件
            },
            btnClick: function(event){
                this.debugLog(event);
            },
            debugLog:function(event){
                console.log(
                    event.srcElement.tagName, 
                    event.srcElement.id, 
                    event.srcElement.innerHTML, 
                    event.key?event.key:""
                );
            },
        },
    });
</script>
```
## 组件
> 组件是可复用的Vue实例,对它是vue实例，也就是vue实例有的属性他都有，但也现有一些区别，组件最终是要渲染成**模板的一部分** 我们在配置组件的时候
一般会配置`template`模板属性

### 全局组件与局部组件
- 全局组件： **直接**使用`Vue.component()`创建的组件，**所有的Vue实例**都可以使用
- 局部组件： 在某个**Vue实例**中注册**只有自己**能使用的组件
```javascript
 // 全局组件
 Vue.component('mycomponent',{
    template: `<div>这是一个自定义组件</div>`,
    data () {
      return {
        message: 'hello world'
      }
    }
  })

 <div id="app">
    <mycomponent></mycomponent>    //引用全局组件
    <my-component></my-component>  //引用自己的局部组件
</div>
<script>
  var app = new Vue({
    el: '#app',
    data: {
    },
    components: {
      'my-component': {
        template: `<div>这是一个局部的自定义组件，只能在当前Vue实例中使用</div>`,
      }
    }
  })
</script>
```
### 模板属性 
> 组件中的模板属性**只能**有一个根元素,下面案例是不被允许的
```javascript
template: `<div>这是一个局部的自定义组件，只能在当前Vue实例中使用</div>
            <button>hello</button>`,  
```

### 组件中的data必须是函数
```javascript
  // data 属性的配置方法
    data () {
      return {
        message: 'hello world'
      }
    }
```
### 属性Props和事件
> 解决父子组件通信问题 父传子 通过`Props属性` 子传父 通过`事件`  
- **Props属性** vue组件通过`props`属性来声明一个自己的属性，然后父组件就可以向该变量传递自己`data`属性的值 
- **事件** 当子组件需要将自己的属性传递给父 就通过**自定义事件**来把这个事情涉及到的数据暴露出来，供父组件处理
```html
<my-component v-bind:foo="baz" v-on:event-a="doThis(arg1,...arg2)"></my-component>
 ```
 - `foo`是<my-component>组件内部定义的一个`prop`属性，`baz`是父组件的一个data属性，
 - `event-a`是子组件定义的一个事件，`doThis`是父组件的一个方法
书写格式：
 - v-bind:foo(子组件props属性)="baz(父组件的data属性)"
 - v-on:event-a(子组件方法事件)="doThis(arg1,...arg2)(父组件的方法)"
过程如下：
- 父组件把`baz`数据通过`prop`传递给子组件的`foo`；
- 子组件内部得到`foo`的值，就可以进行相应的操作；
- 当子组件内部发生了一些变化，希望父组件能知道时，就利用代码触发`event-a`事件，把一些数据发送出去
- 父组件把这个事件处理器绑定为`doThis`方法，子组件发送的数据，就作为`doThis`方法的参数被传进来
- 然后父组件就可以根据这些数据，进行相应的操作   
  
**Props属性代码:**
```javascript
<div id="myApp">
    <div>请输入您的名字：<input v-model="myname"></div>
    <hr/>
    <say-hello :pname="myname" />     //将父组件的myname data属性传递给子组件pname props属性
</div>
<script>
    Vue.component('say-hello', {
        props: ['pname'],
        template: '<div>你好，<strong>{{pname}}</strong>！</div>',
    });
    var myApp = new Vue({
        el: '#myApp', 
        data: {
            myname: 'Koma'
        }
    });
</script>  
```
**自定义事件代码**
```javascript
<div id="app3">
    <my-component2 v-on:myclick="onClick"></my-component2>
</div>
<script>
  Vue.component('my-component2', {
    template: `<div>
    <button type="button" @click="childClick">点击我触发自定义事件</button>
    </div>`,
    methods: {
      childClick () {
        this.$emit('myclick', '这是我暴露出去的数据', '这是我暴露出去的数据2')   //利用$emit来触发
      }
    }
  })
  new Vue({
    el: '#app3',
    methods: {
      onClick () {
        console.log(arguments)
      }
    }
  })
</script>
```
触发步骤如下：   
1.子组件在自己的方法中将自定义事件以及需要发出的数据通过以下代码发送出去
   ```javascript
   this.$emit('myclick', '这是我暴露出去的数据', '这是我暴露出去的数据2')
   ```
   - 第一个参数是自定义事件的名字  对应 v-on:**myclick**(对应这个)="onClick"
   - 后面的参数是依次想要发送出去的数据
2. 父组件利用`v-on`为事件绑定处理器
```javascript
  <my-component2 v-on:myclick="onClick"></my-component2>
```
这样,在Vue实例的methods方法中就可以调用传进来的参数了

## vuex
> vuex是vue框架中全局状态管理插件，全局状态管理，即我们需要定义的全局变量 核心组件包含`state` `getters` `Mutations` `Actions` `Module`
`state`又是核心，定义了哪些全局变量，getters是解决全局变量获取的问题，即通过getters方法获取全局变量(state中的定义)，我们当然也会有修改的
变量值的需求，即通过`Mutations`来修改，但是该方法是同步的，而`Actions`却是异步的，于是我们可以通过`Actions`方法调用`Mutations`来达到最终
修改去全局变量，当我们项目变的越来越大的时候，我们就需要分成不同的`module`方便管理，这是自然而然的思维，这与类的思维一模一样.
### state
```javascript
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
 const state={//要设置的全局访问的state对象
     showFooter: true,
     changableNum:0
     //要设置的初始属性值
   };
 const store = new Vuex.Store({   //将state属性引入到store中
       state
    });
 
export default store;   //导出以便于其他地方导入使用
```
上面案例只有`state`属性，该属性就用来定义全局变量，其他组件导入`store`后，可以通过`this.store.state.showFoote`r或t`his.store.state.changebleNum`在任何一个组件里面获取showfooter和changebleNum定义的值了,但这不是推荐的方式，推荐使用`getter`方法获取state中变量的值，就像类不推荐直接访问变量一样

### getters 
> 通过`getters`属性获取`state`中定义的变量值,该getters可实时监听state值的变化，类似`computed`计算属性,实现双向绑定 通过`this.$store.getters.isShow`方式访问
```javascript
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
 const state={   //要设置的全局访问的state对象
     showFooter: true,
     changableNum:0
     //要设置的初始属性值
   };
const getters = {   //实时监听state值的变化(最新状态)
    isShow(state) {  //方法名随意,主要是来承载变化的showFooter的值
       return state.showFooter
    },
    getChangedNum(){  //方法名随意,主要是用来承载变化的changableNum的值
       return state.changebleNum
    }
};
const store = new Vuex.Store({
       state,
       getters
});
export default store;
```
### mutations
> 当我们定义个全局变量后，很多时候也会有修改的需求，而不仅仅只是访问，而`mutations`就负责修改`state`中全局变量 通过`this.store.commit('show')` 其中`show`是方法名的方式调用  方法定义的时候必须包含`state`参数 因为你是要操作state的
```javascript
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
 const state={   //要设置的全局访问的state对象
     showFooter: true,
     changableNum:0
     //要设置的初始属性值
   };
const getters = {   //实时监听state值的变化(最新状态)
    isShow(state) {  //承载变化的showFooter的值
       return state.showFooter
    },
    getChangedNum(){  //承载变化的changebleNum的值
       return state.changableNum
    }
};
const mutations = {
    show(state) {   //自定义改变state初始值的方法，这里面的参数除了state之外还可以再传额外的参数(变量或对象);
        state.showFooter = true;
    },
    hide(state) {  //同上
        state.showFooter = false;
    },
    newNum(state,sum){ //同上，这里面的参数除了state之外还传了需要增加的值sum
       state.changableNum+=sum;
    }
};
 const store = new Vuex.Store({
       state,
       getters,
       mutations
});
export default store;
```
### actions
> 由于`mutations`是同步操作，而`actions`却是异步方法，所以我们实际中使用`actions`去调用`mutations`设置属性值 方法中必须包含`context`参数
```javascript
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
 const state={   //要设置的全局访问的state对象
     showFooter: true,
     changableNum:0
     //要设置的初始属性值
   };
const getters = {   //实时监听state值的变化(最新状态)
    isShow(state) {  //承载变化的showFooter的值
       return state.showFooter
    },
    getChangedNum(){  //承载变化的changebleNum的值
       return state.changableNum
    }
};
const mutations = {
    show(state) {   //自定义改变state初始值的方法，这里面的参数除了state之外还可以再传额外的参数(变量或对象);
        state.showFooter = true;
    },
    hide(state) {  //同上
        state.showFooter = false;
    },
    newNum(state,sum){ //同上，这里面的参数除了state之外还传了需要增加的值sum
       state.changableNum+=sum;
    }
};
 const actions = {
    hideFooter(context) {  //自定义触发mutations里函数的方法，context与store 实例具有相同方法和属性
        context.commit('hide');
    },
    showFooter(context) {  //同上注释
        context.commit('show');
    },
    getNewNum(context,num){   //同上注释，num为要变化的形参
        context.commit('newNum',num)
     }
};
  const store = new Vuex.Store({
       state,
       getters,
       mutations,
       actions
});
export default store;
```
通过`dispatch` `this.$store.dispatch('hideFooter')`来调用

### module
> 当项目比较大的时候，全局变量都放到一个文件里，显然不行，通常我们会根据不同的应用划分不同的模块，最后再组装起来 于是这种`module
index.js文件里的配置
```javascript
import Vue from 'vue';
import Vuex from 'vuex';
import footerStatus from './modules/footerStatus'
import collection from './modules/collection'
Vue.use(Vuex);

export default new Vuex.Store({
    modules:{
         footerStatus,
         collection
    }
});
```
namespaced: true的含义就是可通过当前模块的名字访问该模块
```
//footerStatus.js
 
const state={   //要设置的全局访问的state对象
     showFooter: true,
     changableNum:0
     //要设置的初始属性值
   };
const getters = {   //实时监听state值的变化(最新状态)
    isShow(state) {  //承载变化的showFooter的值
       return state.showFooter
    },
    getChangedNum(){  //承载变化的changebleNum的值
       return state.changableNum
    }
};
const mutations = {
    show(state) {   //自定义改变state初始值的方法，这里面的参数除了state之外还可以再传额外的参数(变量或对象);
        state.showFooter = true;
    },
    hide(state) {  //同上
        state.showFooter = false;
    },
    newNum(state,sum){ //同上，这里面的参数除了state之外还传了需要增加的值sum
       state.changableNum+=sum;
    }
};
 const actions = {
    hideFooter(context) {  //自定义触发mutations里函数的方法，context与store 实例具有相同方法和属性
        context.commit('hide');
    },
    showFooter(context) {  //同上注释
        context.commit('show');
    },
    getNewNum(context,num){   //同上注释，num为要变化的形参
        context.commit('newNum',num)
     }
};
export default {
    namespaced: true, //用于在全局引用此文里的方法时标识这一个的文件名 
    state,
    getters,
    mutations,
    actions
}
```
###  mapState,mapGetters,mapActions 特殊属性
> 这三个属性本质上减少代码冗余量的 `map`映射作用
```
<script>
 import {mapState} from "vuex"; // 引入mapState 
 export default {
　　　　　　// 下面这两种写法都可以
  computed: mapState({
   count: state => state.count // 组件内的每一个属性函数都会获得一个默认参数state, 然后通过state 直接获取它的属性更简洁  
   count: 'count'　　　　　　　　　// 'count' 直接映射到state 对象中的count, 它相当于 this.$store.state.count,
  })
 }
</script>
```

## Application Structure

> 如下是推荐的应用结构
├── index.html
├── main.js
├── api
│   └── ... # abstractions for making API requests
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # where we assemble modules and export the store
    ├── actions.js        # root actions
    ├── mutations.js      # root mutations
    └── modules
        ├── cart.js       # cart module
        └── products.js   # products module



### 参考
- [VueJS中学习使用Vuex详解](https://segmentfault.com/a/1190000015782272)
- [vuex官网](https://vuex.vuejs.org/guide/modules.html)

## vue-router
> vue-router是vue.js的官方路由器，路由器就是route/view的映射关系，即通过path路径返回对应的视图页面 在vue中是`path--->component'的关系
我们需要做的就是 将组件 (components) 映射到路由 (routes)，然后告诉 Vue Router 在哪里渲染它们

### 起步 
> `<router-link>` 指定path  `<router-view>`路由出口 即对应组件渲染的地方
```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```
`vue-router`在js中的完整配置
- 导入vue-router模块
- 定义路由组件
- 定义路由 路由与组件绑定
- 创建路由实例 
- 创建并将路由挂载到根实例

```javascript
// 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)

// 1. 定义 (路由) 组件。
// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
// 我们晚点再讨论嵌套路由。
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  routes // (缩写) 相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')
```
### 动态路由匹配
> 我们经常需要把**某种模式**匹配到的所有路由，**全都映射**到同个组件  可通过`/user/:id`方式实现
```javascript
const User = {
  template: '<div>User</div>'
}

const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})
```
`/user/foo` 和 `/user/bar` 都将映射到相同的路由 其中":"标记着该字段是一个参数 参数都存放在`$route.params`字段里
|模式|匹配路径|$route.params|
|----|-----|-----|
|/user/:username|/user/evan|{ username: 'evan' }|
|/user/:username/post/:post_id|	/user/evan/post/123|{ username: 'evan', post_id: '123' }|
通过`$route.params.username` 取到对应的`evan`值

### 路由嵌套  
> 路由嵌套通过`children`实现
```html
<div id="app">
  <router-view></router-view>
</div>
```javascript
// 定义user组件 该组件里面嵌套了`router-view`
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}
```
UserProfile 会被渲染在user的<router-view>中
```javascript
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功 
          // UserProfile 会被渲染在 **User 的 <router-view> **中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```
### 命名路由
> 可通过名字的方式访问路由，代替`path`的方式
```js
  const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})  
```
```html
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```
### 命名视图
> 有时候我们想要同级展示多个视图，而不是路由视图，这个时候只需要给`router-view`增加`name`属性，并在路由中，配置多个组件就可以了
```html
 // add name  
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```  
```javascript
  // 同一个路径映射多个组件
  const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
  ```
### 参考资料 
[vue-router官方](https://router.vuejs.org/zh/guide/)

## axios
> axios是http请求客户端，功能非常丰富 如`异步`，自动转换数据，`Intercept`拦截器等功能
### get请求
```js
const axios = require('axios');

// Make a request for a user with a given ID
axios.get('/user?ID=12345')
  .then(function (response) {
    // handle success   Any status code that lie within the range of 2xx cause this function to trigger
     console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.headers);
    console.log(response.config);
  })
  .catch(function (error) {
    // handle error   Any status code that fall outside the range of 2xx  cause this function to trigger
    console.log(error);
  })
  .then(function () {
    // always executed
  });
```
### Intercept
```js
// Add a request interceptor 
axios.interceptors.request.use(function (config) {
    // Do something before request is sent
    return config;
  }, function (error) {
    // Do something with request error
    return Promise.reject(error);
  });
```
全局请求参数配置等其他信息见官方文档，非常详细 [axios官方文档](https://github.com/axios/axios)

## js-cookie
> A simple, lightweight JavaScript API for handling cookies
### set cookie
Create a cookie, valid across the entire site:
```js
Cookies.set('name', 'value')
```
Create a cookie that expires 7 days from now, valid across the entire site:
```js
Cookies.set('name', 'value', { expires: 7 })
```
Create an expiring cookie, valid to the path of the current page:
```js
Cookies.set('name', 'value', { expires: 7, path: '' })   
```
### read cookie
```js
Cookies.get('name') // => 'value'
Cookies.get('nothing') // => undefined 
Cookies.get() // => { name: 'value' }  // Read all visible cookies:
```
更多详细信息参见官方文档：[js-cookie官方文档](https://github.com/js-cookie/js-cookie)

## path-to-regexp 
针对路径字符串进行处理的正则表达式工具 使用见官网[path-to-regexp](https://github.com/pillarjs/path-to-regexp)

## nprogress
一款基于JavaScript的进度条UI组件 使用参考官网[nprogress](https://github.com/rstacruz/nprogress)


## npm
> npm 前端`javascript`包管理器

### 设置npm源
```js
# 设置为淘宝的镜像源
npm config set registry https://registry.npm.taobao.org
# 设置为官方镜像源
npm config set registry https://registry.npmjs.org
```
### package.json
> 包管理文件  主要包含依赖包和作者信息
- scripts字段 :npm run *   //* 是sricpts中的键值 比如 dev  build:prod 
- `dependencies` 生产环境依赖
- `devDependencies` 开发环境依赖
```js
{
  "name": "xxxx",
  "description": "xxxx",
  "scripts": {
    "dev": "vue-cli-service serve",
    "build:prod": "vue-cli-service build",
    "svgo": "svgo -f src/icons/svg --config=src/icons/svgo.yml"
  },
  "dependencies": {
    "axios": "0.18.1",
  },
  "devDependencies": {
    "@babel/core": "7.0.0",
  }
```
### 常用命令
```js
npm install <package_name> 
npm isntall <git url>
 
#安装最新版本
npm install <package_name>@latest
 
#安装指定版本
npm install <package_name>@0.1.1

#安装写到dependencies,-s是简写
npm install <package_name> --save
npm install <package_name> --s
 
#安装写到devDependencies,-D是简写,
npm install <package_name> --save-dev
npm install <package_name> --D
 
#安装beta版本
npm install <module-name>@beta
 
#npm更新
npm update <package_name>
 
#npm 卸载
npm uninstall <package_name>
 
#npm执行脚本，package.json文件有个scripts字段，可以定义脚本命令(lint,build)，npm直接执行
npm run lint
npm run build
 
#npm run 执行script下面所有的命令
npm run
 
#dev 脚本，开发阶段要做的处理.dev是自定义命令
npm run dev 
 
#serve,脚本用于启动服务,serve是自定义命令
npm run serve
```
参考资料：[npm详解](https://blog.csdn.net/xingmeiok/article/details/90299089)


## vue项目启动流程
写的非常详细:[webpack项目启动流程](https://www.cnblogs.com/xifengxiaoma/p/9493544.html)

## vue常用插件
`vetur` `eslint`(syntax check) `prettier`(format) `vue-devtools`(chrome debug)
## 参考资料
> 每篇参考文章都很经典 值得仔细阅读
- [深刻理解Vue中的组件](https://segmentfault.com/a/1190000010527064)
- [vue官网](https://cn.vuejs.org/v2/guide/)
- [如何在5天内学会Vue？聊聊我的学习方法！](https://mp.weixin.qq.com/s/R--Qh4Lp5nhhO0eNWNDmIw)
- [VueJS中学习使用Vuex详解](https://segmentfault.com/a/1190000015782272)
- [vuex官网](https://vuex.vuejs.org/guide/modules.html)
