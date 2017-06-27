此文研究页面中的图片资源的加载和渲染时机，使得我们能更好的管理图片资源，避免不必要的流量和提高用户体验。

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
<img src="../image/pink.png" style="display:none">
<div class="img-purple" style="display:none"></div>
```
图片资源请求如下：
![oru7 h3rz qu3snw _umf4a](https://user-images.githubusercontent.com/9698086/27082582-f2380d3a-5077-11e7-9878-e218c8e0c04f.png)

设置了`display:none`属性的元素，图片不会渲染出来，但会加载。

```
<style>
.img-yellow {
	background-image: url(../image/yellow.png);
}
</style>
<div style="display:none">
	<img src="../image/red.png">
	<div class="img-yellow"></div>
</div>
```
图片资源请求如下：
![6cu vs 5 ix xddy9b6e qx](https://user-images.githubusercontent.com/9698086/27082391-18d9f472-5077-11e7-8618-1dab27fbef5d.png)

设置了`display:none`属性元素的子元素，样式表中的背景图片不会渲染出来，也不会加载；而`<img>`标签的图片不会渲染出来，但会加载。

### 重复图片

```
.img-blue {
	background-image: url(../image/blue.png);
}
<div class="img-blue"></div>
<img src="../image/blue.png">
<img src="../image/blue.png">
```
图片资源请求如下：
![ekje2 4t d 6fy b 7](https://user-images.githubusercontent.com/9698086/27082367-0386e10c-5077-11e7-9ed5-7ade17ddb994.png)

页面中多个`<img>`标签或样式表中的背景图片图片地址是同一个，图片只加载一次。

### 不存在元素的背景图片
```
.img-blue {
	background-image: url(../image/blue.png);
}
.img-orange{
	background-image: url(../image/orange.png);
}
<div class="img-orange"></div>
```
图片资源请求如下：
![sg_8k tuwn6a yd4w g qr](https://user-images.githubusercontent.com/9698086/27082444-510752d6-5077-11e7-9d2a-5dee39ec0511.png)

不存在元素的背景图片不会加载。
这是因为背景图片只会加载渲染树上存在的，渲染树是通过把DOM树和样式规则树匹配产生的，所以DOM树上没有的元素，在渲染树不会存在。

### 伪类的背景图片
```
.img-green {
	background-image: url(../image/green.png);
}
.img-green:hover{
	background-image: url(../image/red.png);
}
<div class="img-green"></div>
```
触发hover前的图片资源请求如下：
![c4 ix2_x 0s na7h _md u](https://user-images.githubusercontent.com/9698086/27082522-be2a30d6-5077-11e7-8edd-12da92cdd0e4.png)

触发hover后的图片资源请求如下：
![w_vorv 7 x6fy75 q4n _a5](https://user-images.githubusercontent.com/9698086/27082557-daca1d46-5077-11e7-91c0-48aa65d585aa.png)

当触发伪类的时候，伪类样式上的背景图片才会加载。

## 应用

### 占位图

当使用样式表中的背景图片作为占位符时，要把背景图片转为base64格式。这是因为背景图片加载的顺序在`<img>`标签后面，背景图片可能会在`<img>`标签图片加载完成后才开始加载，达不到想要的效果。

### 预加载

很多场景里图片是在改变或触发状态后才显示出来的，例如点击一个Tab后，一个设置`display:none`隐藏的父元素变为显示，这个父元素里的子元素图片会在父元素显示后才开始加载；又如当鼠标hover到图标后，改变图标图片，图片会在hover上去后才开始加载，导致出现闪一下这种不友好的体验。

在这种场景下，我们就需要把图片预加载，预加载有很多种方式:
1. 若是小图标，可以合并成雪碧图，在改变状态前就把所有图标都一起加载了。
2. 使用上文讲到的，设置了display:none属性的元素，图片不会渲染出来，但会加载。把要预加载的图片加到设置了`display:none`的元素背景图或`<img>`标签里。
3. 在javascript创建img对象，把图片url设置到img对象的src属性里。