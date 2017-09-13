# JavaScript错误处理

当 JavaScript 引擎执行 JavaScript 代码时，会发生各种错误：
可能是语法错误，通常是程序员造成的编码错误或错别字。
可能是拼写错误或语言中缺少的功能（可能由于浏览器差异）。
可能是由于来自服务器或用户的错误输出而导致的错误。

而 Javascript 引擎是单线程的，因此一旦遇到错误，JavaScript 引擎通常会停止，阻塞后续代码的执行并抛出一个错误信息，因此我们很有必要管理好各种错误异常。

## Try / Catch

**try...catch** 语句将能引发错误的代码放在try块中，并且对应一个响应，然后有异常被抛出。

**try** 语句包含了由一个或者多个语句组成的 **try** 块, 和至少一个catch子句或者一个finally子句的其中一个，或者两个兼有， 下面是三种形式的try声明:

* try...catch
* try...finally
* try...catch...finally

try语句中放入可能会产生错误的语句或函数

catch语句中包含要执行的语句，当try语句中抛出错误时，catch语句会捕捉到这个错误信息，并执行catch语句中的代码，如果在try块中没有异常抛出，这catch子句将会跳过。

finally语句在try块和catch块之后执行。无论是否有异常抛出或着是否被捕获它总是执行。当在finally语句中抛出错误信息时会覆盖掉try语句中的错误信息。

如果从finally块中返回一个值，那么这个值将会成为整个try-catch-finally的返回值，无论是否有return语句在try和catch中。这包括在catch块里抛出的异常。

```
try {
   try_statements
}
[catch (exception) {
   catch_statements
}]
[finally {
   finally_statements
}]
```

## throw

throw 语句用来抛出一个用户自定义的异常。当前函数的执行将被停止（throw之后的语句将不会执行），并且控制将被传递到调用堆栈中的第一个 catch 块。如果调用者函数中没有catch块，程序将会终止。

```
throw expression; 
```

## Promise中的异常

抛出错误
```
new Promise((resolve,reject)=>{
	reject();
})
```
```
Promise.resolve().then((resolve,reject)=>{
	reject();
});
```
```
Promise.reject();
```
```
throw expression; 
```

捕捉错误

```
.catch((exception)=>{
	catch_statements
})
```

集中错误处理

## Error对象

**throw** 和 **Promise.reject** 可以抛出字符串类型的错误，而已可以抛出一个 **Error** 对象类型的错误。

## Try / Catch 性能