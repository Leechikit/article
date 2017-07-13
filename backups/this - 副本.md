## 换个角度看Javascript中的this

在网上很多文章都对Javascript中的this做了详细的介绍，下面我从另一个角度来解释this指向，使大家更好判断。

## this

MDN对**this**的解释是这样的：
> In most cases, the value of this is determined by how a function is called. It can't be set by assignment during execution, and it may be different each time the function is called. 

我们可以总结为：

1. **this**的值取决于函数的调用方式。
2. 每次函数被调用时，**this**的值都会重新赋值。

可以看出，要判断**this**的值是什么，实际上就是判断**this**所在函数是如何被调用的。

## call方法
**call**方法允许切换函数执行的上下文环境（context），即 this 绑定的对象。

当我们使用**call**方法调用函数时，我们能很清晰就能知道调用函数的**this**的值，既是**call**方法的第一个参数。那么如果我们把所有的函数都使用**call**方法调用，就能一目了然**this**的值，要明确**call**方法的第一个参数是什么值并不容易，下面我们从**call**方法的模拟实现来一探究竟：

```
Function.prototype.call = function(thisArg) {
	var context = thisArg || window;
	var arr = [];
	var result;
	context = Object(context);
	context.fn = this;
	for (let i = 1, len = arguments.length; i < len; i++) {
		arr.push('arguments[' + i + ']');
	}
	result = eval("context.fn(" + arr + ")");
	delete context.fn;
	return result;
}
```

**call**方法的模拟实现原理是第一个参数对象创建一个方法，这个方法引用需要调用的函数，并执行这个方法，相当于在已对象方法的形式调用这个函数。

MDN对对象方法中的**this**有以下说明：
> When a function is called as a method of an object, its 'this' is set to the object the method is called on.
> 当一个函数作为对象的方法被调用时，函数的**this**指向了这个对象。

因此函数中的**this**就指向了**call**方法中的第一个参数，这样我们要把所有函数调用方法转换成**call**方法调用就相当于转换成对象方法调用。

## 转换为对象方法调用

下面我们把所以函数的调用方式都转换成对象方法的调用，这样我们就很明确函数中的**this**是指向调用的该方法的对象。

### 函数中的this
```
function foo(){
	console.log(this);
}
foo();
```
可以转换成
```
window.foo = function(){
	console.log(this);
}
window.foo();
```


### 构造函数中的this
```
function Foo(){
	this.print = function(){
		console.log(this);	
	}
}

var foo = new Foo();
foo.print();
```
可以转换成
```
foo.print = function(){
	console.log(this);
}
```

### 闭包中的this
```
var foo = {
	print: function(){
		(function(){
			console.log(this);
		})();
	}
}
foo.print();
```
执行`foo.print()`时，实际执行的是
```
(function(){
	console.log(this);
})();
```
可以转换成
```
window.foo = function(){
	console.log(this);
}
window.foo();
```

### 参数中的this
```
var foo = {
	print: function(){
		console.log(this);
	}
}
setTimeout(foo.print, 1000);
```
因为函数参数是按值传递的，因此`setTimeout`1秒后执行的不止`foo.print`的引用，而是`foo.print`的值，相当于如下：
```
setTimeout(function(){console.log(this)},1000);
// 1秒后执行的函数是
function(){
	console.log(this);
}
// 而不是foo.print
```
可以转换成
```
window.foo = function(){
	console.log(this);
}
window.foo();
```

### DOM事件处理函数中的this

```
DOMElement.addEventListener('click',function(){
	console.log(this);
});
```
可以转换成
```
DOMElement.clickHandle = function(){
	console.log(this);
}
// 点击时执行
DOMElement.clickHandle();
```

## ES6 箭头函数的this

箭头函数的**this**总是指向定义时所在的对象，而不是运行时所在的对象。箭头函数并不会绑定一个**this**变量，它的作用域会如同寻常所做的一样一层层地去往上查找。