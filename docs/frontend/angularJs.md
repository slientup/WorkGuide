### 现在主流的前端是vue，但由于工作中老项目的原因，依然会维护angularJs

`解决问题`：AngularJS主要作用能实现数据的动态刷新 最终都是html中的变量跟控制器中的变量做绑定


### 核心指令  

任何一个AngularJS都`必须拥有至少这三种元素` 
- `ng-app` 指令告诉 AngularJS，某个html元素下内容是AngularJS 应用程序 的"所有者"。  
- `ng-controller` 用于定义一个控制器 该html元素下的变量跟ng-controller中的存在对应关系  
- `Scope` 作用域就是提供应用在 `HTML (视图)` 和 `controller (控制器)`之间的`纽带`  

```
<div ng-app="myApp" ng-controller="myCtrl">  //app下面控制器是myCtrl

名: <input type="text" ng-model="firstName"><br>  
姓: <input type="text" ng-model="lastName"><br>
<br>
姓名: {{firstName + " " + lastName}}

</div>

<script>
var app = angular.module('myApp', []);     
app.controller('myCtrl', function($scope) {    //引入$scope变量 这样在html页面可以直接使用
    $scope.firstName = "John";
    $scope.lastName = "Doe";
});
</script>
```

### AngularJS路由是一种纯前端的解决方案  
他的本质是当请求一个url时，根据路由配置匹配这个url，然后请求模板片段，并插入到ng-view中去，其中模块的内容就是通过控制器来改变

1. 项目中的base.html  
<html lang="en" ng-app="App">  //直接在html页面下就直接属于了angularJs 
    
2. 在App中的路由配置  
```
angular.module("App", [
        "ui.router",
]).config(function($stateProvider, $httpProvider, w5cValidatorProvider){
    $stateProvider.state("erspan", {  
            url: "/erspan.html",  //这是对应的在url中显示的url
            templateUrl: "/static/erspan/templates/erspan.html",   //这是调用对应的模板
            controller: "erspanCtrl"  //这是控制器  
```
3. 控制器的配置   
```
     angular.module("App")
    .controller("erspanCtrl", function($scope){
    })       //该控制器注册到App中，没做任何操作 
```
4. 前端调用url  
```
   <li><a href="/erspan/#/erspan.html"><i class="oicon-target"></i>ERSPAN</a></li>
```

### 注意事项
- AngularJs中有自己独立的标签，在该应用中需使用AngularJs独立的标签
- 当 a href 的url中含有`#`这个标识的时候，就会走 AngularJS路由的模式，如果不包含就是以前普通的url，直接调用对应的url
- ng-view中嵌入的是路由变化后的返回页面 但这个在低版本似乎在
