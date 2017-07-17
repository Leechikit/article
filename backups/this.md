# 换个思路理解Javascript中的this

在网上很多文章都对 **Javascript** 中的 **this** 做了详细的介绍，但大多是介绍各个绑定方式或调用方式下 **this** 的指向，于是我想有一个统一的思路来更好理解 **this** 指向，使大家更好判断，以下有部分内容不是原理，而是一种解题思路。

## 从call方法开始

> **call** 方法允许切换函数执行的上下文环境（**context**），即 **this** 绑定的对象。

大多数介绍 **this** 的文章中都会把 **call** 方法放到最后介绍，但此文我们要把 **call** 方法放在第一位介绍，并从 **call** 方法切入来研究 **this** ，因为 **call** 函数是显式绑定 **this** 的指向，我们来看看它如何模拟实现（不考虑传入 **null** 、 **undefined** 和原始值）：

```
Function.prototype.call = function(thisArg) {
	var context = thisArg;
	var arr = [];
	var result;

	context.fn = this;

	for (let i = 1, len = arguments.length; i < len; i++) {
		arr.push('arguments[' + i + ']');
	}

	result = eval("context.fn(" + arr + ")");

	delete context.fn;

	return result;
}
```

从以上代码我们可以看到，把调用 **call** 方法的函数作为第一个参数对象的方法，此时相当于把第一个参数对象作为函数执行的上下文环境，而 **this** 是指向函数执行的上下文环境的，因此 **this** 就指向了第一个参数对象，实现了 **call** 方法切换函数执行上下文环境的功能。

## 对象方法中的this

在模拟 **call** 方法的时候，我们使用了对象方法来改变 **this** 的指向。调用对象中的方法时，会把对象作为方法的上下文环境来调用。

既然 **this** 是指向执行函数的上下文环境的，那我们先来研究一下调用函数时的执行上下文情况。

下面我门来看看调用对象方法时执行上下文是如何的：
```
var foo = {
	x : 1,
	getX: function(){
		console.log(this.x);
	}
}
foo.getX();
```
![object-method](https://user-images.githubusercontent.com/9698086/28258338-57957528-6b03-11e7-8f86-da2efaa08431.png)

从上图中，我们可以看出`getX`方法的调用者的上下文是`foo`，因此`getX`方法中的 **this** 指向调用者上下文`foo`，转换成 **call** 方法为`foo.getX.call(foo)`。

下面我们把其他函数的调用方式都按调用对象方法的思路来转换。

## 构造函数中的this

```
function Foo(){
	this.x = 1;
	this.getX = function(){
		console.log(this.x);
	}
}
var foo = new Foo();
foo.getX();
```

执行 **new** 如果不考虑原型链，只考虑上下文的切换，就相当于先创建一个空的对象，然后把这个空的对象作为构造函数的上下文，再去执行构造函数，最后返回这个对象。

```
var newMethod = function(func){
	var context = {};
	func.call(context);
	return context;
}
function Foo(){
	this.x = 1;
	this.getX = function(){
		console.log(this.x);
	}
}
var foo = newMethod(Foo);
foo.getX();
```

![creater-method](https://user-images.githubusercontent.com/9698086/28258394-a053ad2a-6b03-11e7-858d-ab0804c0edef.png)

## DOM事件处理函数中的this

```
DOMElement.addEventListener('click', function(){
	console.log(this);
});
```

把函数绑定到DOM事件时，可以当作在DOM上增加一个函数方法，当触发这个事件时调用DOM上对应的事件方法。
```
DOMElement.clickHandle = function(){
	console.log(this);
}
DOMElement.clickHandle();
```
![domelement-method](https://user-images.githubusercontent.com/9698086/28259586-8f855efc-6b09-11e7-9066-6d595297565b.png)

## 普通函数中的this
```
var x = 1;
function getX(){
	console.log(this.x);
}
getX();
```

这种情况下，我们创建一个虚拟上下文对象，然后普通函数作为这个虚拟上下文对象的方法调用，此时普通函数中的**this**就指向了这个虚拟上下文。

> 那这个虚拟上下文是什么呢？在非严格模式下是全局上下文，浏览器里是 **window** ，NodeJs里是 **Global** ；在严格模式下是 **undefined** 。

```
var x = 1;
function getX(){
	console.log(this.x);
}

[viturl context].getX = getX;
[viturl context].getX();
```
![normal-function](https://user-images.githubusercontent.com/9698086/28267650-8fdfdf0c-6b2d-11e7-8229-00fd20bfd531.png)

## 闭包中的this
```
var x = 1;
var foo = {
	x: 2,
    y: 3,
	getXY: function(){
		(function(){
			console.log("x：" + this.x);
			console.log("y：" + this.y); 
		})();
	}
}
foo.getXY();
```
这段代码的上下文如下图：
![closure-function1](https://user-images.githubusercontent.com/9698086/28262511-ce899602-6b15-11e7-8fb2-2938fb41162f.png)

这里需要注意的是，我们再研究函数中的 **this** 指向时，只需要关注 **this** 所在的函数是如何调用的， **this** 所在函数外的函数调用都是浮云，是不需要关注的。因此在所有的图示中，我们只需要关注红色框中的内容。

因此这段代码我们关注的部分只有:
```
(function(){
	console.log(this.x);
})();
```

与普通函数调用一样，创建一个虚拟上下文对象，然后普通函数作为这个虚拟上下文对象的方法立即调用，匿名函数中的 **this** 也就指向了这个虚拟上下文。
![closure-function2](https://user-images.githubusercontent.com/9698086/28262501-c3b1ad0a-6b15-11e7-8119-4a994fd06b7d.png)

## 参数中的this
```
var x = 1;
var foo = {
	x: 2,
	getX: function(){
		console.log(this.x);
	}
}
setTimeout(foo.getX, 1000);
```

函数参数是值传递的，因此上面代码等同于以下代码：
```
var getX = function(){
	console.log(this.x);
};
setTimeout(getX, 1000);
```

然后我们又回到了普通函数调用的问题。

## 全局中的this

全局中的 **this** 指向全局的上下文

```
var x = 1;
console.log(this.x);
```
![global-this](https://user-images.githubusercontent.com/9698086/28262537-e4d11778-6b15-11e7-8006-d26f0f61219c.png)

## 复杂情况下的this

```

```

## 总结

在需要判断 **this** 的指向时，我们可以安装这种思路来理解：

* 判断 **this** 在全局中OR函数中，若在全局中则 **this** 指向全局，若在函数中则只关注这个函数并继续判断。
* 判断 **this** 所在函数是否作为对象方法调用，若是则 **this** 指向这个对象，否则继续操作。
* 创建一个虚拟上下文，并把**this**所在函数作为这个虚拟上下文的方法，此时 **this** 指向这个虚拟上下文。
* 在非严格模式下虚拟上下文是全局上下文，浏览器里是 **window** ，Node.js里是 **Global** ；在严格模式下是 **undefined** 。

图示如下：
![judge-this](https://user-images.githubusercontent.com/9698086/28267532-0b00edf8-6b2d-11e7-91b4-05bb966c0544.png)
