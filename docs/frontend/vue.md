##  vue现在最流行的前端框架 
> vue是前端框架，是目前最接近后端编程语言思路的框架，对后端程序员来说上手很容易

- [vue基础](#vue基础)
- [vue组件](#vue组件)
- [vuex](#vuex)
- [vue-router](vue-router)
- [axios](#axios)
- [js-cookie](#js-cookie)
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
### watch计算属性
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

## 参考资料
- [深刻理解Vue中的组件](https://segmentfault.com/a/1190000010527064)
- [如何在5天内学会Vue？聊聊我的学习方法！](https://mp.weixin.qq.com/s/R--Qh4Lp5nhhO0eNWNDmIw)
