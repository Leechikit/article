# 事件委托中的阻止冒泡

今天同事小英童鞋又问了我一个问题：

*html*：
```
<div class="container">
	<ul class="list">
		<li class="item">
			<span class="button"></span>
		</li>
	</ul>
</div>
```

*javascript*
```
$('.container').on('click', '.button', (event) => {
	event.stopPropagation();
	console.log('click button');
});

$('.list').on('click', '.item', (event) => {
	console.log('click item');
});

// 点击<span class="button"></span>
// click button
// click item
```

当点击`<span class="button"></span>`时，控制台输出`click button`和`click item`，阻止`span.button`中的事件冒泡失败了。