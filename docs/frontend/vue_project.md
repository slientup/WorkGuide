[vue+element_ui实战视频](https://www.bilibili.com/video/BV1EE411B7SU?from=search&seid=17791471872295886821)
> 该视频是目前看过最好的vue实战视频，从vue搭建、用户登录token认证、element-ui各种组件的使用，模型如何布局，都一步一步详解简单明了。

- 该项目实例: http://shop.liulongbin.top/
- 前端代码:https://gitee.com/wBekvam/vue-shop-admin.git
- 后端代码:https://gitee.com/wBekvam/vueShop-api-server.git
- 页面预览:http://shop.liulongbin.top/

### `slot-scope`
在table中常用`slot-scope`来获取当前行的信息

### mounted和created
mounted()方法在dom元素创建之后调用，而created函数在dom元素创建之前调用，当函数里面要调用dom元素的时候，使用mounted方法，一般使用第三方的组件
### 修改模型样式
如何修改样式? 我们通过`element-ui`引入的元素，一般会有对应的`class`，通过f12查到对应的class 然后再对该class进行修改就行      
如果没有对应的class 那自己在命名一个
### 进度条
npgress进度条实现的最页面上方加载的进度的显示 在拦截器中实现，在请求的时候展示进度条 在`response`中隐藏进度条
### 移除console.log
`babel-plugin-transfrom-remove-console`  只在发布阶段移除console.log
### 优化打包大小
核心思路:如果包提供了cdn源，就使用cdn源  externals 加载外部cdn资源 优化打包大小通过cdn优化elementui的组件打包
但要注意在企业内部开发，可能无法访问cdn资源，需将包下载到本地


在搭建前端项目结构时，主要参考`vue-admin-template` or `mall-admin-web`,而在使用中对某些组件的使用不熟练的话(涵盖了常用的组件)，则可参考本视频学习，涵盖表单验证，
tab，message提示，多级菜单，`多级选择框` `对话框`，`搜索框`，`tag`，`进度条`，`echarts`和富文本编辑器`vue-quill-editor` `时间线` `进度条` `折叠显示` `上传图片`等等

**总结** 在`element-ui`组件的选择的过程中，都是看到自己适合的组件，然后复制主体样例代码，保留关键的，然后再一点一点的添加**需要的**功能，多看看这个项目。

多看优秀的项目，优秀的组件，在需要的时候将其组件的代码复用过来就行.


