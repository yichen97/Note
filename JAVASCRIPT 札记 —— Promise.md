# JAVASCRIPT 札记 

#  Promise

## 一、 概述

### 1. Promise  的含义

​	为了解决回调函数和事件造成的回调地狱，更加方便的**异步**编程方案。

​	所谓`Promise`，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

### 2.  Promise 的特点

​	1> 有三种状态pending（进行中）、fulfilled（已成功）、rejected（已失败）。状态只受其中异步操作的影响。

​	2> 当结果发生（已成功或者已失败）状态不会再变。之后添加的回调函数也只会得到这个结果。

### 3. Promise 的缺点

​	1> 无法取消，建立就会执行无法中途取消。

​	2> 如果不设置回调函数，Promise中的错误不会反应到外部。

​	3> pending状态下，内部执行进度未知。

## 二、 基本用法

​	ES6 规定，`Promise`对象是一个构造函数，用来生成`Promise`实例。

```javascript
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

​	`Promise`构造函数接受一个函数作为参数，该函数的两个参数`resolve`、`reject`也是两个函数（由 `javascript` 引擎提供）。

​	`resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；`reject`函数的作用是，将`Promise`对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

​	`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数。

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

​	`then`方法可以接受两个回调函数作为参数。第一个回调函数是`Promise`对象的状态变为`resolved`时调用，第二个回调函数是`Promise`对象的状态变为`rejected`时调用。这两个函数都是可选的，不一定要提供。它们都接受`Promise`对象传出的值作为参数。

​	下面是一个`Promise`对象的简单例子。

```javascript
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```

​	上面代码中，`timeout`方法返回一个`Promise`实例，表示一段时间以后才会发生的结果。过了指定的时间（`ms`参数）以后，`Promise`实例的状态变为`resolved`，就会触发`then`方法绑定的回调函数。

下面是异步加载图片的例子。

```javascript
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    const image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}
```



