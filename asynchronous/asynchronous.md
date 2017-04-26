# 异步编程的那些事

## 前言

## 异步与同步

## javascript的异步机制

* 回调函数callback（ajax，setTimeout和setInterval)
* 事件监听
* 发布/订阅
* Promise函数
* Generator函数
* async函数

## 使用javascript的异步机制控制动画
有这样一个需求，要依次请求多个接口，每个接口返回后执行对应的动画，每个动画也是依次执行。执行的逻辑入下图：