# es6特性
> 记录一些es特性的用法

## 几乎与对象.方式访问属性

## =>箭头函数语法
```
function(x){
	reuturn x+6;
}
等价于
(x)=>x+6;
或者
x=>x+6;
```

## const简写
```js
const {dispatch} = this.props;
等价于
const dispatch = this.props.dispatch;
```
## filter 过滤器
> 这与java的filter含义一样
```js
var arr = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];
const newArr = arr.filter(word => word.length > 6);
console.log(newArr);   //["exuberant", "destruction", "present"]
```
## promise对象
Promise是异步编程的一种解决方案，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。
从语法上说，Promise是一个对象，从它可以获取异步操作的消息。

## export和export default的区别
一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用`export`关键字输出该变量 
```js
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
// 另外文件导入 必须制定具体的名字
import { lastName as surname } from './profile.js';
```
`export default`不指定具体的名字，在import的时候可**自行命名**
```js
// export-default.js
export default function () {
  console.log('foo');
}
// import-default.js
import customName from './export-default';  //contomName 自定义的名字
customName(); // 'foo'  
```
参考资料:[export 和 export default 的区别](https://www.cnblogs.com/fanyanzhao/p/10298543.html)

## ==和===的区别
> `==`用于一般比较，`===`用于严格比较，`==`在比较的时候可以转换数据类型，`===`严格比较，只要类型不匹配就返回flase



