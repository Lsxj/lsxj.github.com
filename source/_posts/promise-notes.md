---
title: 一道笔试题引发的promise笔记   
date: 2016-08-04 17:54:01
categories: 前端开发
tags: [es6, promise, javascript]
---
### 前言
近来参加校招笔试，发现有好几道关于Promise的题目。然而我都没有了解过。所以，这篇文章以网易笔试的一道题开始，记录关于Promise的那些事。

### 笔试题

	console.log(1);
	new Promise(function (resolve, reject){
	    reject(true);
	    window.setTimeout(function (){
	        resolve(false);
	    }, 0);
	}).then(function(){
	    console.log(2);
	}, function(){
	    console.log(3);
	});
	console.log(4);


请问输出结果是什么？在揭晓答案之前，我们还是先来了解一下Promise吧。

### Promise是什么
> Promise 对象用于异步(asynchronous)计算.。一个Promise对象代表着一个还未完成，但预期将来会完成的操作。
> Promise 对象是一个返回值的代理，这个返回值在promise对象创建时未必已知。它允许你为异步操作的成功或失败指定处理方法。 这使得异步方法可以像同步方法那样返回值：异步方法会返回一个包含了原返回值的 promise 对象来替代原返回值。

### Promise的API
#### Constructor
使用`new`来调用`Promise`的构造器进行实例化

	var promise = new Promise(function(resolve, reject){
	    //异步处理
	    //处理结束后，调用resolve或reject
	});


#### Instance Method
对通过new生成的promise对象为了设置其在resolve(成功)/reject(失败)时调用的回调函数可以使用`promise.then()`实例方法


	promise.then(onFulfilled, onRejected);
	//以防误解，上述的即为以下这种形式
	promise.then(function(){...}, function(){...});


当resolve(成功)时，会调用`onFulfilled`函数；
reject(失败)时，会调用`onRejected`函数。
这也对应了前面笔试题中,`onFulfilled`其实就是`console.log(2)`，也就是说成功时会调用`console.log(2)`，而失败时，`onRejected`就是调用`console.log(3)`。

若只想处理异常情况的函数，可`promise.then(undefined, onRejected)`，当然更好的选择是用`promise.catch()`来处理。二者效果相同。


	promise.then(function (value) {
	    console.log(value); 
	}).catch(function (error) {
	    console.log(error);
	});
	
	//等同于以下形式
	promise.then(function (value) {
	    console.log(value);
	}, function (error) {
	    console.log(error);
	});


#### 其他方法
* `Promise.all()`
* `Promise.race()`
* `Promise.resolve()`
* `Promise.reject()`

### Promise的状态
Promise对象有三种状态
* pending 初始状态，既不是fulfilled也不是rejected
* fulfilled 成功。此时调用`onFulfilled`
* rejected 失败。此时调用`onRejected`

Fulfilled和Rejected都可以表示为`Settled`。
由下图可以了解，最初Promise为`pending`状态，在执行后转为`settled`状态，而`settled`状态分为两种：在成功后转为`fulfilled`，执行`.then(onFulfilled)`方法；在失败后转为`reject`，执行`.then(onRejecttion)`或`.catch(onRejecttion)`，进行异步操作，再返回Promise对象，转为`pending`状态。

![Promise状态图](http://oao50r2ex.bkt.clouddn.com/image/blog0804_1.png)

### `Promise.resolve` & `Promise.reject`
#### Promise.resolve
`Promise.resolve(value)`可认为是`new Promise()`方法的快捷方式

	Promise.value(value);
	
	//等同于以下代码
	
	new Promise(function(resolve){
	    resolve(value);
	});


此时这个promise对象会进入`fulfilled`状态。而`resolve(value)`中的`value`会传递给后面`then`中指定的`onFulfilled`函数。

`Promise.resolve(value)`返回值也是一个promise对象，所以可以进行链式调用.

#### Promise.reject
`Promise.reject(error)`与上述静态方法类似，也是`new Promise()`方法的快捷方式


	Promise.reject(new Error("出错了"));
	//等同于以下代码
	new Promise(function(resolve, reject){
	    reject(new Error("出错了"));
	});

这段代码则是调用该promise对象通过`then`指定的`onRejected`函数，并将错误对象(Error)传递给`onRejected`函数。

### 笔试题解答
话不多说，贴图就是。

![答案截图](http://oao50r2ex.bkt.clouddn.com/image.blog0804_2.png)

可以知道，当promise调用了`reject(true)`方法，则传递`true`这个参数给'then'指定的`onRejected`函数，即题目中的`function(){console.log(3);}`。但由于'.then'中指定的方法调用是**异步执行**的，所以会先执行console.log(4);

本篇文章仅是简单介绍promise。欲了解更多内容，可查看以下资料。谢谢~

资料来源：
* [Promise- Javascipt | MDN][1]
* [JavaScript Promise迷你书（中文版)][2]


  [1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
  [2]: http://liubin.org/promises-book/