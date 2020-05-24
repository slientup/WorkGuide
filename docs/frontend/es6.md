# es6特性
> 记录一些es特性的用法
## const简写
```js
const {dispatch} = this.props;
等价于
const dispatch = this.props.dispatch;
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
[参考资料](https://www.cnblogs.com/fanyanzhao/p/10298543.html)



