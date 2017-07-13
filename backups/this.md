# 换个角度看Javascript中的this

## 从call方法开始

大多数介绍**this**的文章中都会把**call**方法放到最后介绍，但此文我们要把 **call** 方法放在第一位介绍，并从 **call** 方法切入来研究**this**。

> **call**方法允许切换函数执行的上下文环境（context），即 this 绑定的对象。

## 对象方法中的this