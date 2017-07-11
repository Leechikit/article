# HTML5拖放API Drag and Drop

此文研究Web API中的拖放接口，提供各个属性和方法的说明，解决拖放过程中的拖拽数据对象存储和获取问题。

拖放API作用到两个目标对象，分别是拖拽目标对象和放置目标对象。


## 拖拽目标

一个设置`draggable`属性的值为`true`DOM元素或者一个选中状态的文本区块可以成为拖拽目标。

```
<div draggable="true"></div>
```
OR
![yy 20170629201338](https://user-images.githubusercontent.com/9698086/27686981-a7a89b4e-5d07-11e7-8a8a-b3c86431d290.jpg)

## 放置目标

一个绑定了下图放置目标对应的5个事件的DOM元素可以成为放置目标。

## 事件

拖放API有8个事件，其中有3个事件绑定在拖拽目标上，有5个事件绑定在放置目标上。

### 绑定在拖拽目标
| Evnet        | Description                                       |
| ------------ | ------------------------------------------------- |
| dragstart    | 当用户开始拖拽一个元素或者一个文本选取区块的时触发。|
| drag         | 当用户正在拖拽一个元素或者一个文本选取区块的时触发。|
| dragend      | 当用户结束拖拽一个元素或者一个文本选取区块的时触发。（如放开鼠标按键或按下键盘的 escap 键）|

### 绑定在放置目标
| Event        | Description                                       |
| ------------ | ------------------------------------------------- |
| dragenter    | 当一个元素或文字选取区块被拖曳移动进入一个有效的放置目标时触发。|
| dragover     | 当一个元素或文字选取区块被拖曳移动经过一个有效的放置目标时触发。|
| dragleave    | 当一个元素或文字选取区块被拖曳移动离开一个有效的放置目标时触发。|
| dragexist    | 当一个元素不再是被选取中的拖曳元素时触发。（Firefox能触发，触发顺序：dragexist->dragleave->drop；Chrome无法触发）|
| drop         | 当一个元素或文字选取区块被放置至一个有效的放置目标时触发。|

通过下图能更直观观察每个事件触发的时机

![dragdrop-events](https://user-images.githubusercontent.com/9698086/27691241-0b6b902e-5d16-11e7-8689-e07e13f9a654.gif)

[戳我看源码](https://jsfiddle.net/leechikit/7buhdcko/)

> 注意：在**dragover**事件中使用`event.preventDefault();`阻止默认事件，才能触发**drop**事件

## DataTransfer对象
在进行拖放操作时，会触发上面所述的8个事件，每个**event**事件对象中都会有**DataTransfer**对象用来保存被拖动的数据。它可以保存一项或多项数据、一种或者多种数据类型。

### effectAllowed
用来指定拖动时被允许的效果。

在**dragstart**事件中设置

### 属性

### dropEffect
设置实际的放置效果，它应该始终设置成**effectAllowed**的可能值之一 。

在**dragenter**事件和**dragover**事件中设置


effectAllowed和dropEffect属性的栗子：[戳我看源码](https://jsfiddle.net/leechikit/o6r3wtpq/)

### files
包含一个在数据传输上所有可用的本地文件列表。如果拖动操作不涉及拖动文件，此属性是一个空列表。

```
filesZoneEl.addEventListener("drop", (event) => {
	event.preventDefault();
	let files = event.dataTransfer.files;
	for (let i = 0, len = files.length; i < len; i++) {
		let liEl = document.createElement("li");
		liEl.innerHTML = files[i].name;
		filesListEl.appendChild(liEl);
	}
});
```

![drag-file](https://user-images.githubusercontent.com/9698086/27758455-bb7be73a-5e46-11e7-93ea-993f217e0cdf.gif)

[戳我看源码](https://jsfiddle.net/leechikit/dqgjr4r0/)

### types
保存一个被存储数据的类型列表作为第一项，顺序与被添加数据的顺序一致。如果没有添加数据将返回一个空列表。

### items
存储**DataTransferItem**数据对象的列表。

### 方法

### addElement()
设置拖动源。
```
event.dataTransfer.addElement(element);
```

### setData()
为一个给定的类型设置数据并存储在**items**属性中。

### getData()
从**items**属性中获取给定类型的数据，无数据时返回空字符串。
```
event.dataTransfer.getData(type);
```

### clearData()
从**items**属性中删除与给定类型关联的数据，若类型为空则删除所有数据。
```
event.dataTransfer.clearData(type);
```

### setDragImage()
自定义一个期望的拖动时的图片，默认为被拖动的节点。
```
event.dataTransfer.setDragImage(imgElement, offsetX, offsetY);

```
| Param     | Description                   |
| --------- | ----------------------------- |
| imgElement| 要用作拖动反馈图像元素。|
| offsetX   | 图像内的水平偏移量。|
| offsetY   | 图像内的垂直偏移量。|

设置拖动时的图片时，要把图片预加载，否则图片会在拖动开始**dragstart**事件触发时才会加载图片，会导致拖动图出不来或闪一下的后果。可把图片放到`<img>`标签并设置`display:none;`，原理详看我之前的文章[Web图片资源的加载与渲染时机](https://segmentfault.com/a/1190000010032501)。

![drag-imgage](https://user-images.githubusercontent.com/9698086/27691268-1908df7a-5d16-11e7-8ab6-076982c21c38.gif)

## DataTransferItemList
**dataTramsfer**对象的**items**属性，包含了一系列**DataTransferItem**拖拽数据对象。

### 属性

### length
数组长度。

### 方法

### add()
增加一个拖拽数据对象到**items**属性中，并返回增加的拖拽数据对象。
```
event.dataTransfer.items.add(file);
```

### remove()
从**items**属性中移除指定位置的一个拖拽数据对象。
```
event.dataTransfer.items.remove(index);
```

### clear()
清空**items**属性中的所拖拽数据对象。
```
event.dataTransfer.items.clear();
```

## DataTransferItem
**DataTransferItemList**列表中的拖拽数据对象。

### 属性

### kind
拖拽数据对象类型。

| Value     | Description                   |
| --------- | ----------------------------- |
| file      | 文件类型。|
| string    | 文本字符串类型。|

### type
MIME类型的Unicode字符串，例如**text/plain**、**text/html**或**image/png**。

### 方法

### getAsFile()
若拖拽数据对象是文件类型，则返回一个文件对象。
```
let itemList = event.dataTransfer.items;
for (let i = 0, len = itemList.length; i < len; i++) {
	if (itemList[i].kind == "file") {
		console.log(itemList[i].getAsFile());
	}
}
```

### getAsString()
若拖拽数据对象是文本字符串类型，通过回调函数获取拖拽数据中的字符串数据。
```
let itemList = event.dataTransfer.items;
for (let i = 0, len = itemList.length; i < len; i++) {
	if (itemList[i].kind == "string") {
		itemList[i].getAsString((data) => {
			console.log(data);
		});
	}
}
```

## 拖放对象的数据存储

在进行拖放操作时，有可能需要把拖拽目标的数据传送给放置目标，此时一般操作是在**dragstart**事件触发时把需要的数据存储到一个变量，然后再**drop**事件触发时获取这个变量。但当**dragstart**事件和**drop**事件在不同的文件定义，又不想玷污全局变量的情况下，我们需要更好的办法来存储拖放数据。

在**DataTransfer**对象中的**items**属性就是用来存储拖放数据的，数据类型分为文本类型和文件类型。

### 存储文本字符串类型数据：
```
event.dataTransfer.setData(type, data);
```
OR
```
event.dataTransfer.items.add(data, type);
```
> 一种文本字符串类型只能存储一个数据，当重复文本字符串类型存储数据时，后者会覆盖前者。

### 存储文件类型数据：
```
event.dataTransfer.items.add(file);
```

### 获取所有文本字符串类型的拖拽数据对象
```
event.dataTransfer.types
```

### 获取所有文件类型的拖拽数据对象
```
let files = event.dataTransfer.files;
for (let i = 0, len = files.length; i < len; i++) {
	console.log(files[i]);
}
```
OR
```
let itemList = event.dataTransfer.items;
for (let i = 0, len = itemList.length; i < len; i++) {
	if (itemList[i].kind == "file") {
		console.log(itemList[i].getAsFile());
	}
}
```

### 获取所有文本字符串类型的拖拽数据对象
```
let itemList = event.dataTransfer.items;
for (let i = 0, len = itemList.length; i < len; i++) {
	if (itemList[i].kind == "string") {
		itemList[i].getAsString((data) => {
			console.log(data);
		});
	}
}
```

### 获取指定文本字符串类型的拖拽数据对象
```
event.dataTransfer.getData(type);
```

### 删除指定文本字符串类型的拖拽数据对象
```
event.dataTransfer.clearData(type);
```

### 删除指定位置的拖拽数据对象
```
event.dataTransfer.items.remove(index);
```

### 清空所有拖拽数据对象
```
event.dataTransfer.clearData();
```
OR
```
event.dataTransfer.items.clear();
```

### 栗子

上面的几个栗子都使用了以上方法存储和获取拖拽数据对象，感兴趣的可以看看源码。

> 欢迎关注：[Leechikit](https://segmentfault.com/u/leechikit/articles)
> 原文链接：[segmentfault.com](https://segmentfault.com/a/1190000010032501)
>
> 到此本文结束，欢迎提问和指正。
> 写原创文章不易，若本文对你有帮助，请点赞、推荐和关注作者支持。

