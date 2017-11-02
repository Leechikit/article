# 事件委托中的阻止冒泡

今天同事小英童鞋又问了我一个问题：

*html*：
```
<div id="outer"">
    <div id="inner1">
        <div id="inner2">
            <div id="inner3">
            </div>
        </div>
    </div>
</div
```

*javascript*
```
$('#outer').on('click', '#inner3', event => {
    console.log('click inner3');
});

$('#inner1').on('click', '#inner2', event => {
    event.stopPropagation();            
    console.log('click inner2');
});

// 点击<div id="inner3"></div>
// click inner3
// click inner2
```

当点击`<div id="inner3"></div`时，控制台输出`click inner3`和`click inner2`，阻止`<div id="inner3"></div`中的事件冒泡失败了。

## 事件流
我们先来了解一下DOM的事件流。

事件流有三个阶段：
1. 捕捉阶段(Capturing)：事件将沿着DOM树向下转送，目标节点的每一个祖先节点，直至目标节点。
2. 目标阶段(target)：浏览器在查找到已经指定给目标事件的事件监听器之后，就会运行 该事件监听器。目标节点就是触发事件的DOM节点。
3. 冒泡阶段(Bubbling)：事件将沿着DOM树向上转送，再次逐个访问目标元素的祖先节点到document节点。

在 **event** 事件中的 **eventPhase** 属性用来记录事件流的阶段，使用`event.eventPhase`可以很清晰的看到事件流传递到各个元素中事件流处于哪个阶段。

[戳我看源码](https://codepen.io/leechikit/pen/mqbGbd)

事件传播方式有事件冒泡和事件捕获。

DOM标准处理事件时会按先捕捉传递后冒泡传递的顺序传递事件，所以事件会先后进过捕捉阶段，目标阶段，冒泡阶段三种阶段。

对于事件目标上的事件监听器来说，事件会处于“目标阶段”，而不是冒泡阶段或者捕获阶段。在目标阶段的事件会触发该元素（即事件目标）上的所有监听器，而不在乎这个监听器到底在注册时useCapture 参数值是true还是false。

从顺序上讲，捕获式绑定的事件处理方法要优先于冒泡式绑定的事件处理方法，不过这只是针对元素作为父结点的时候，当元素是激活目标时，各种方式绑定的事件处理方法被执行顺序不确定的，是按照绑定顺序进行。

![image](/https://user-images.githubusercontent.com/9698086/32093167-fca43796-bb2d-11e7-8b7b-a7dd28bcb385.png)

## 事件委托

当我们需要对很多元素添加事件的时候，可以通过将事件添加到它们的父节点而将事件委托给父节点来触发处理函数。

[戳我看源码](https://codepen.io/leechikit/pen/YEKONP)

一般的事件委托都是在冒泡阶段完成。

```
const outerEl = document.getElementById('outer');
outerEl.addEventListener('click', clickOuterHandle, false);

function clickOuterHandle(event) {
    let target = event.target;
    if (target.id == 'inner2') {
        alert('click inner2');
    }
}
```

在上面的代码中，我们把点击`div#inner2`触发的事件委托到`div#outer`上。

点击`div#inner2`后，事件从沿着DOM树向下传送到事件目标，在这个捕捉阶段中并没有发现有监听器，因此不会触发任何事件，然后再沿着DOM树向上传送，在冒泡阶段阶段发现`div#outer`上有监听器，触发这个监听器，`event.target`指向事件起源目标，即`div#inner2`，判断后触发`alert('click inner2')`。

## 阻止冒泡