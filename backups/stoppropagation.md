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

事件的传递顺序有两种：
1. 捕捉
2. 冒泡

事件流分别有三个阶段：
1. 捕捉阶段(Capturing)：
2. 目标阶段(target)：
3. 冒泡阶段(Bubbling)：

DOM标准处理事件时会按先捕捉传递后冒泡传递的顺序传递事件，所以事件会先后进过捕捉阶段，目标阶段，冒泡阶段三种阶段。

## 事件委托

## 阻止冒泡