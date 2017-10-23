# 傻傻分不清的__proto__与prototype 

同事小英今天问了我一个问题：

```
function Myname(firstName, lastName){
	this.firstName = firstName;
	this.lastName = lastName; 
}
Myname.prototype.logName = function(){
	Myname.combineName();
	console.log(this.fullName);
}
Myname.prototype.combineName = function(){
	this.fullName = `${this.firstName} ${this.lastName}`
}

var myname = new Myname('Sanfeng', 'Zhang');
myname.logName(); // Uncaught TypeError: Myname.combineName is not a function
```

小英认为`Myname`的原型`Myname.prototype`，所以`Myname`会继承`Myname.prototype`的属性，调用`Myname.combineName()`相当于调用`Myname.prototype.combineName()`，但结果`Myname.combineName()`不是一个方法。