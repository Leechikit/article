# 傻傻分不清的__proto__与prototype

今天同事小英童鞋问了我一个问题：

```
function Foo(firstName, lastName){
	this.firstName = firstName;
	this.lastName = lastName; 
}
Foo.prototype.logName = function(){
	Foo.combineName();
	console.log(this.fullName);
}
Foo.prototype.combineName = function(){
	this.fullName = `${this.firstName} ${this.lastName}`
}

var foo = new Foo('Sanfeng', 'Zhang');
foo.logName(); // Uncaught TypeError: Foo.combineName is not a function
```

小英童鞋认为`Foo`的原型对象是`Foo.prototype`，所以`Foo`会继承`Foo.prototype`的属性，调用`Foo.combineName()`相当于调用`Foo.prototype.combineName()`，但结果`Foo.combineName()`不是一个方法。

会造成这个问题的原因一定是因为小英童鞋弄混了原型和继承的一些原理，下面我们来整理一下原型和继承的相关原理，找出问题的根本原因。

## `prototype`

`prototype`是一个只有函数才有的属性。

当创建函数时，**JavaScript** 会为这个函数自动添加`prototype`属性，这个属性指向的是一个原型对象`Functionname.prototype`。我们可以向这个原型对象添加属性或对象，甚至可以指向一个现有的对象。

## `__proto__`

接下来我们说说继承，每个对象都有一个`__proto__`属性，这个属性是用来标识自己所继承的原型。

## 原型链

**JavaScript** 可以通过`prototype`和`__proto__`在两个对象之间创建一个关联，使得一个对象就可以通过委托访问另一个对象的属性和函数。

这样的一个关联就是原型链，一个由对象组成的有限对象链，用于实现继承和共享属性。

## 构造函数创建对象实例

**JavaScript** 函数有两个不同的内部方法：**[[Call]]** 和 **[[Construct]]** 。

如果不通过`new`关键字调用函数，则执行 **[[Call]]** 函数，从而直接执行代码中的函数体。

当通过`new`关键字调用函数时，执行的是 **[[Construct]]** 函数，它负责创建一个实例对象，把实例对象的`__proto__`属性指向构造函数的`prototype`来实现继承构造函数`prototype`的所有属性和方法，将`this`绑定到实例上，然后再执行函数体。

模拟一个构造函数：
```
function createObject(proto) {
	if (!(proto === null || typeof proto === "object" || typeof proto === "function"){
        throw TypeError('Argument must be an object, or null');
    }
	var obj = new Object();
	obj.__proto__ = proto;
	return obj;
}

var foo = createObject(Foo.prototype);
```

至此我们了解了`prototype`和`__proto__`的作用，也了解使用构造函数创建对象实例时这两个属性的指向，以下使用一张图来总结一下如何通过`prototype`和`__proto__`实现原型链。

![proto](https://user-images.githubusercontent.com/9698086/32085949-a3399e04-bb04-11e7-9f4f-b4051108b5da.png)

从上图我们可以找出`foo`对象和`Foo`函数的原型链：
```
foo.__proto__ = Foo.prototype;
foo.__proto__.__proto__ = Foo.prototype.__proto__ = Object.prototype;
foo.__proto__.__proto__.__proto__ = Foo.prototype.__proto__.__proto__ = Object.prototype.__proto__ = null;
```

![foo](https://user-images.githubusercontent.com/9698086/32086657-57c098a6-bb09-11e7-8789-6c4dce13df2a.png)

```
Foo.__proto__ = Function.prototype;
Foo.__proto__.__proto__ = Function.prototype.__proto__;
Foo.__proto__.__proto__.__proto__ = Function.prototype.__proto__.__proto__ = Object.prototype.__proto__ = null;
```

![class-foo](https://user-images.githubusercontent.com/9698086/32086658-57e8bdae-bb09-11e7-8479-2a769b4ffbc6.png)

构造函数`Foo`的原型链上没有`Foo.prototype`，因此无法继承`Foo.prototype`上的属性和方法。而实例`foo`的原型链上有`Foo.prototype`，因此`foo`可以继承`Foo.prototype`上的属性和方法。

到这里，我们可以很简单的解答小英童鞋的问题了，在`Foo`的原型链上没有`Foo.prototype`，无法继承`Foo.prototype`上的`combineName`方法，因此会抛出`Foo.combineName is not a function`的异常。要想使用`combineName`方法，可以这样`Foo.prototype.combineName`，或者这样`this.combineName`（`this`指向实例对象）。
