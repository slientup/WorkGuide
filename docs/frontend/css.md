# 记录css中常用的一些样式
- [常用命令](#常用命令)
  

- [基础知识](#基础知识)

## 常用命令
```css 
  // 盒子位置相关的配置
  width: 450px;   
  height: 360px;
  background-color: #fff;
  border-radius: 3px;   //边角的长度
  position: absolute;   
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);  //对元素进行平移  
  background-color: #fff;
  bottom: 60px;   // 底部距离
  padding: 0 20px;  
  cursor: pointer;    //鼠标变成小手
  
  // 字体相关的配置
  font-size: 10px;
  line-height: 24px;  
  color: #fff;    // 字体颜色
  text-align: center;  //居中
  letter-spacing: 0.2em;  //间隔 
```

css层级配置样式 **针对元素**
```css      
.el-header {
  background-color: #373f41;
  > div {   
    display: flex;
    img {
      height: 40px;
    }
    span {
      margin-left: 15px;
    }
  }
}
```
css层级配置样式 **针对class自定义**
```css
.login_box {
  background-color: #fff;
  .avatar_box {
    width: 200px;
    img {
      width: 100%;
    }
  }
}
```
```
<style lang="less" scoped>
/* // lang="less" 支持less格式
// scoped vue的指令，只在当前组件生效 */
```

**建议**：前端组件css样式修饰的时候：优先使用`ui组件`提供的样式布局(`Layout` `Container`)，如果没有相应的样式再利用上面的自定义.


## 基础知识

## 块级元素和行内元素
### 分类
- **块级元素block**：**独占一行** 对宽高属性生效，如果不给宽度，块级元素默认宽度是`100%`  

- **行内元素(inline)**: 可以多个标签存在一行，**宽高属性值不生效**，完全**靠内容**撑开宽高 

- **行内块元素(inline-block)**：结合的行内和块级的优点，既可以**设置长宽**，可以让`padding`和`margin`生效，又可以和其他**行内元素并排**

![分类](https://segmentfault.com/img/bVbbW3s?w=547&h=158/view)
### 元素转换
 - 块级元素转换为行内元素：`display:inline`  
 - 行内元素转换为块级元素：`display:block`  
 - 转换为行内块元素：`display：inline-block`  
### 使用场景
块级元素: 一般用于布局  行内元素 一般用于内容

## 盒子模型
> **所有html元素**都可以看做是盒模型
![盒模型](https://www.runoob.com/images/box-model.gif)
```css
div {
    width: 300px;   内容的width宽度  
    border: 25px solid green;
    padding: 25px;
    margin: 25px;
}
```
实际宽度：300px (宽)+ 50px (左 + 右填充)+ 50px (左 + 右边框)+ 50px (左 + 右边距)= 450px 

## flex模型
> 盒状模型布局是传统的布局方案，它对于那些特殊布局非常不方便，比如，垂直居中就不容易实现，而`flex`就是解决这类问题，默认ui组件的容器模型就是`flex`
任何一个容器都可以指定为 Flex 布局
```css
.box{
  display: flex;
}
// 行内元素的实现方式
.box{
  display: inline-flex;
}
```
### 容器属性
- flex-direction  决定项目的排列方向,默认主轴`水平`方向，起点在左端
- flex-wrap  默认情况下，项目都排在一条线（又称"轴线"）上。`flex-wrap`属性定义，如果一条轴线排不下，如何换行
- flex-flow  `flex-direction`属性和`flex-wrap`属性的简写
- justify-content  定义项目在主轴上的**对齐方式**
- align-items  定义项目在交叉轴上(**垂直方向**)如何对齐
- align-content  多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用

更详细全面的信息参考：[Flex布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

## 网页布局
### 相对定位
> 生成相对定位的元素，相对于其**正常位置（普通流）**进行定位
### 绝对定位
> 相对于 static 定位以外的第一个**父元素**进行定位
### 固定定位
> 相对于**浏览器窗口**（而绝对定位相对于父元素），其余的特点类似于绝对定位    `top 50%` `left 50%`  就能自动定位到中间了







