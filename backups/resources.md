## 浏览器的工作流程
要研究图片资源的加载和渲染，我们先要了解浏览器的工作原理。以**Webkit**引擎的工作流程为例：

![webkitflow](https://user-images.githubusercontent.com/9698086/26868233-de8d0f06-4b9a-11e7-8b35-0c6bfbe9871b.png)

从上图可看出，浏览器加载一个HTML页面后进行如下操作：

* 解析HTML —> 产生DOM树
* 加载样式 —> 解析样式 —> 产生样式规则树
* 把DOM树和样式规则树匹配产生渲染树
* 计算元素位置进行布局
* 绘制展示

从上图我们不能很直观的看出图片资源从什么时候开始加载，下面我们先给出结论，告诉大家图片加载和渲染的时机：

* 解析HTML【遇到`<img>`标签加载图片】 —> 产生DOM
* 加载样式 —> 解析样式【遇到图片链接不加载】 —> 产生样式规则树
* 把DOM树和样式规则树匹配产生渲染树【加载渲染树上的图片】
* 计算元素位置进行布局
* 绘制展示【开始渲染图片】

## 图片加载规则
页面中不是所有的`<img>`标签图片和样式表背景图片都会加载。

### **display:none**

```
<style>
.img-purple {
	background-image: url(../image/purple.png);
}
</style>
<img src="..image/pink.png" style="display:none">
<div class="img-purple" style="display:none"></div>
```

设置了`display:none`属性的元素，图片不会渲染出来，但会加载。

```
<style>
.img-yellow {
	background-image: url(../image/yellow.png);
}
</style>
<div style="display:none">
	<img src="..image/red.png">
	<div class="img-yellow"></div>
</div>
```

设置了`display:none`属性元素的子元素，图片不会渲染出来，也不会加载。

### 重复图片

```
.img-blue {
	background-image: url(../image/blue.png);
}
<div class="img-blue"></div>
<img src="../image/blue.png">
<img src="../image/blue.png">
```

页面中多个`img`标签或样式表中的背景图片图片地址是同一个，图片只加载一次。

### 不存在元素的背景图片
```
.img-blue {
	background-image: url(../image/blue.png);
}
<div class="img-blue"></div>
<div class="img-orange"></div>
```

不存在元素的背景图片不会加载。
这是因为背景图片只会加载渲染树上存在的，渲染树是通过把DOM树和样式规则树匹配产生的，所以DOM树上没有的元素，在渲染树不会存在。

### 伪类的背景图片
```
.img-orange {
	background-image: url(../image/orange.png);
}
.img-orange:hover{
	background-image: url(../image/red.png);
}
<div class="img-orange"></div>
```

当触发伪类的时候，伪类样式上的背景图片才会加载。

## 应用

###占位图

当使用样式表中的背景图片作为占位符时，要把背景图片转为base64格式。这是因为背景图片加载的顺序在`<img>`标签后面，背景图片可能会在`<img>`标签图片加载完成后才开始加载，达不到想要的效果。
