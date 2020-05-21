##  vue现在最流行的前端框架 
> vue是前端框架，是目前最接近后端编程语言思路的框架，对后端程序员来说上手很容易

- [vue基础](#vue基础)
- [vuex](#vuex)
- [vue-router](vue-router)
- [axios](#axios)
- [js-cookie](#js-cookie)

### vue基础
> vue属性是vue最重要的特性之一，其中**数据**和**方法**属性最重要

**VUE实例**

#### data数据属性
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

#### method方法属性
参考上面举例的`reversedMessage()`就是vue中方法属性的定义方式

#### computed计算属性
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
#### watch计算属性
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
#### filters过滤器属性
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
#### 钩子方法
每个 Vue 实例在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等
钩子方法的作用就是在实例被创建前或者后**自动执行**   
常用钩子方法如下：`created`、`mounted`、`updated`和`destroyed` 其中`mounted`的方法通常初始化.


#### 常用指令
> 这部分主要是html中的指令
- `v-on` 绑定事件  如v-on:click 单击事件
- `v-model` 绑定vue实例中data属性 **只能用于form表单框** 
- `v-if`  `v-show` `v-for`

#### 事件
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
### 组件
> 组件是可复用的Vue实例,对它是vue实例，也就是vue实例有的它都有



参考资料：
- [如何在5天内学会Vue？聊聊我的学习方法！](https://mp.weixin.qq.com/s/R--Qh4Lp5nhhO0eNWNDmIw)
