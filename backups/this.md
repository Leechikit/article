# 换个角度看Javascript中的this

## 从call方法开始

大多数介绍**this**的文章中都会把**call**方法放到最后介绍，但此文我们要把 **call** 方法放在第一位介绍，并从 **call** 方法切入来研究**this**。

> **call**方法允许切换函数执行的上下文环境（context），即 this 绑定的对象。

现在我们来模拟**call**方法
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

从以上代码我们可以看到，把调用**call**方法的函数作为第一个参数对象的方法，此时相当于把第一个参数对象作为函数执行的上下文环境，而**this** 是指向函数执行的上下文环境的，因此**this**就指向了第一个参数对象，实现了**call**方法切换函数执行上下文环境的功能。

## 对象方法中的this

在模拟**call**方法的时候，我们使用了对象方法来改变**this**的指向。调用对象中的方法时，会把对象作为方法的上下文环境来调用。

既然**this**是指向执行函数的上下文环境的，那我们先来研究一下调用函数时的执行上下文情况。

执行上下文包括变量对象、作用域链和**this**指针，这里我们主要关注变量对象和**this**指针。

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

[图]

从上图中，我们可以看出`getX`方法的调用者的上下文是`foo`，因此`getX`方法中的**this**指向调用者上下文`foo`。

下面我们把其他函数的调用方式都按调用对象方法的思路来转换，并最终转换成**call**函数。

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

执行**new**如果不考虑原型链，只考虑上下文的切换，就相当于先创建一个空的对象，然后把这个空的对象作为构造函数的上下文，再去执行构造函数，最后返回这个对象。

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

[图]

## 普通函数中的this
```
var x = 1;
function getX(){
	console.log(this.x);
}
getX();
```

[图]

## 闭包中的this
```
var x = 1;
var foo = {
	x: 2,
	getX: function(){
		(function(){
			console.log(this.x);
		})();
	}
}
foo.getX();
```

[图]

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

[图]

## DOM事件处理函数中的this
```
DOMElement.clickHandle = function(){
	console.log(this);
}
```

[图]

## 全局中的this
```
var x = 1;
console.log(this.x);
```

[图]

