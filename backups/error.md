# JavaScript错误处理


当 **JavaScript** 引擎执行 **JavaScript** 代码时，有可能会发生各种错误，例如是语法错误，语言中缺少的功能，或由于来自服务器或用户的错误输出而导致的错误。

而 **Javascript** 引擎是单线程的，因此一旦遇到错误，**Javascript** 引擎通常会停止执行，阻塞后续代码并抛出一个错误信息，因此对于可预见的错误，我们应该捕捉并正确展示给用户或开发者。

## Try / Catch

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

**try** 语句包含了由一个或者多个语句组成的 **try** 块, 和至少一个 **catch** 子句或者一个 **finally** 子句的其中一个，或者两个兼有， 下面是三种形式的 **try** 声明:

* try...catch
* try...finally
* try...catch...finally

**try** 语句中放入可能会产生错误的语句或函数

**catch** 语句中包含要执行的语句，当 **try** 语句中抛出错误时，**catch** 语句会捕捉到这个错误信息，并执行 **catch** 语句中的代码，如果在 **try** 块中没有异常抛出，这 **catch** 子句将会跳过。

**finally** 语句在 **try** 块和 **catch** 块之后执行。无论是否有异常抛出或着是否被捕获它总是执行。当在 **finally** 语句中抛出错误信息时会覆盖掉 **try** 语句中的错误信息。

```
try {
    try {
        throw new Error('can not find it1');
    } finally {
        throw new Error('can not find it2');
    }
} catch (err) {
    console.log(err.message);
}

// can not find it2
```

如果从 **finally** 块中返回一个值，那么这个值将会成为整个 **try-catch-finally** 的返回值，无论是否有 **return** 语句在 **try** 和 **catch** 中。这包括在 **catch** 块里抛出的异常。

```
function test() {
    try {
        throw new Error('can not find it1');
        return 1;
    } catch (err) {
        throw new Error('can not find it2');
        return 2;
    } finally {
        return 3;
    }
}

console.log(test()); // 3
```

## Throw

```
throw expression; 
```

**throw** 语句用来抛出一个用户自定义的错误。当前函数的执行将被停止（**throw** 之后的语句将不会执行），并且控制将被传递到调用堆栈中的第一个 **catch** 块。如果调用者函数中没有 **catch块**，程序将会终止。

```
try {
    console.log('before throw error');
    throw new Error('throw error');
    console.log('after throw error');
} catch (err) {
    console.log(err.message);
}

// before throw error
// throw error
```

## Promise中的异常

**Promise** 中抛出错误的方法：
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

**Promise** 中捕捉错误的方法：

```
promiseObj.then(undefined, (err)=>{
	
});
```

```
promiseObj.catch((exception)=>{
	catch_statements
})
```

在 **JavaScript** 函数中，只有 **return** / **yield** / **throw** 会中断函数的执行，其他的都无法阻止其运行到结束的。

在 **resolve** / **reject** 之前加上 **return** 能阻止往下继续运行。

without `return`：
```
Promise.resolve()
.then(() => {
    console.log('before excute reject');
    reject(new Error('throw error'));
    console.log('after excute reject');
})
.catch((err) => {
    console.log(err.message);
});

// before excute reject
// throw error
// after excute reject
```

use `return`：
```
Promise.resolve()
.then(() => {
    console.log('before excute reject');
    return reject(new Error('throw error'));
    console.log('after excute reject');
})
.catch((err) => {
    console.log(err.message);
});

// before excute reject
// throw error
```

### Throw or Reject

无论是 **try...catch** 还是 **promise** 都能捕获到的是“同步”错误

**reject** 是回调，而 **throw** 只是一个同步的语句，如果在另一个异步的上下文中抛出，在当前上下文中是无法捕获到的。

因此在 **Promise** 中使用 **reject** 抛出错误。否则 **catch** 有可能会捕捉不到。

```
Promise.resolve()
.then(() => {
    setTimeout(()=>{
        throw new Error('throw error');
    },0);
})
.catch((err) => {
    console.log(err);
});

// Uncaught Error: throw error
```

```
Promise.resolve()
.then(() => {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            reject(new Error('throw error'));
        }, 0);
    });
})
.catch((err) => {
    console.log(err);
});

// Error: throw error
```

## Error对象

**throw** 和 **Promise.reject** 可以抛出字符串类型的错误，而已可以抛出一个 **Error** 对象类型的错误。

创建自己的错误构造函数

```
function MyError(message = 'Default Message') {
    this.name = 'MyError';
    this.message = message;
    this.stack = (new Error()).stack;
}
MyError.prototype = Object.create(Error.prototype);
MyError.prototype.constructor = MyError;

export default MyError;
```

在代码中抛出自定义的错误类型并捕捉

```
try {
	throw new MyError("some message");
} catch(e){
	console.log(e.name + ":" + e.message);
}
```

## 统一错误处理

代码中抛出的异常，一种是要展示给用户，一种是展示给开发者。

对于展示给用户的错误，一般使用 **alert** 或 **toast** 展示；对于展示给开发者的错误，一般输出到控制台。

在一个函数或一个代码块中可以把抛出的错误统一捕捉起来，按照不同的错误类型以不同的方式展示，对于。

*ensureError.js*
```
function EnsureError(message = 'Default Message') {
    this.name = 'EnsureError';
    this.message = message;
    this.stack = (new Error()).stack;
}
EnsureError.prototype = Object.create(Error.prototype);
EnsureError.prototype.constructor = EnsureError;

export default EnsureError;
```

*toastError.js*
```
function ToastError(message = 'Default Message') {
    this.name = 'ToastError';
    this.message = message;
    this.stack = (new Error()).stack;
}
ToastError.prototype = Object.create(Error.prototype);
ToastError.prototype.constructor = ToastError;

export default ToastError;
```

*errorHandler.js*
```
import EnsureError from './ensureError.js';
import ToastError from './toastError.js';
import EnsurePopup from './ensurePopup.js';
import ToastPopup from './toastPopup.js';

function errorHandler(err) {
    if (err instanceof EnsureError) {
        EnsurePopup(err.message);
    } else if (err instanceof ToastError) {
		ToastPopup(err.message);
	}else {
        console.error(err);
    }
}

export default errorHandler;
```

## Try / Catch 性能
