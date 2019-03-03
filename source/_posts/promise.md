---
title: ES6 Promise学习笔记
date: 2017-03-04
tags: ['promise']
categories: ['笔记']
---
## ES6 Promise
1. [Promise的由来及含义](#a1)
2. [Promise基本用法](#a2)
3. [Promise.prototype.then()](#a3)
4. [Promise.prototype.catch()](#a4)
5. [Promise.all()](#a5)
6. [Promise.race()](#a6)
7. [Promise.resolve()](#a7)
8. [Promise.reject()](#a8)
9. [应用](#a9)
10. [参考资料](#a10)
<!--more-->
***
## <a id='a1'>1.Promise的由来及含义</a>
一直以来，JavaScript处理异步都是以callback的方式，在前端开发领域callback机制几乎深入人心。在设计API的时候，不管是浏览器厂商还是SDK开发商亦或是各种类库的作者，基本上都已经遵循着callback的套路。

在callback的模型里边，我们假设需要执行一个异步队列，代码看起来可能像这样：
````javascript
loadImg('a.jpg', function() {
    loadImg('b.jpg', function() {
        loadImg('c.jpg', function() {
            console.log('all done!');
        });
    });
});
````

近几年随着JavaScript开发模式的逐渐成熟，CommonJS规范顺势而生，其中就包括提出了Promise规范，Promise完全改变了js异步编程的写法，让异步编程变得十分的易于理解。

Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。Promise对象有以下两个特点。

（1）对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：Pending（进行中）、Resolved（已完成，又称 Fulfilled）和Rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从Pending变为Resolved和从Pending变为Rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。
## <a id='a2'>2.Promise基本用法</a>
Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。

下面代码创造了一个Promise实例。
````javascript
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
````
resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从Pending变为Resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从Pending变为Rejected）

Promise实例生成以后，可以用then方法分别指定Resolved状态和Reject状态的回调函数(可选)。这两个函数都接受Promise对象传出的值作为参数。
````javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
````
## <a id='a3'>3.Promise.prototype.then()</a>
then方法是定义在原型对象Promise.prototype上的，因此Promise实例具有then方法。它的作用是为Promise实例添加状态改变时的回调函数。then方法的第一个参数是Resolved状态的回调函数，第二个参数（可选）是Rejected状态的回调函数。

then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。
````javascript
getJSON("/post/1.json").then(
  post => getJSON(post.commentURL)
).then(
  comments => console.log("Resolved: ", comments),
  err => console.log("Rejected: ", err)
);
````
## <a id='a4'>4.Promise.prototype.catch()</a>
Promise.prototype.catch方法是.then(null, rejection)的别名，用于指定发生错误时的回调函数。

Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。
````javascript
getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
````
跟传统的try/catch代码块不同的是，如果没有使用catch方法指定错误处理的回调函数，Promise对象抛出的错误不会传递到外层代码，即不会有任何反应。(Chrome浏览器不遵守这条规定)
## <a id='a5'>5.Promise.all()</a>
Promise.all方法用于将多个Promise实例，包装成一个新的Promise实例，类似“与”关系。
````javascript
var p = Promise.all([p1, p2, p3]);
````
p的状态由p1、p2、p3决定，分成两种情况。

（1）只有p1、p2、p3的状态都变成resolve，p的状态才会变成resolve，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

（2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。
## <a id='a6'>6.Promise.race()</a>
Promise.race方法同样是将多个Promise实例，包装成一个新的Promise实例，类似“或”关系。
````javascript
var p = Promise.race([p1, p2, p3]);
````
只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。
## <a id='a7'>7.Promise.resolve()</a>
Promise.resolve()可将现有对象转为Promise对象。

Promise.resolve方法的参数分成四种情况。

（1）参数是一个Promise实例

原封不动地返回这个实例。

（2）参数是一个thenable对象

Promise.resolve方法会将这个对象转为Promise对象，然后就立即执行thenable对象的then方法。

（3）参数不是具有then方法的对象，或根本就不是对象

返回一个新的Promise对象，状态为Resolved。

（4）不带有任何参数

直接返回一个Resolved状态的Promise对象。
## <a id='a8'>8.Promise.reject()</a>
Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected.

Promise.reject()方法的参数，会原封不动地作为reject的理由，变成后续方法的参数。

````javascript
const thenable = {
  then(resolve, reject) {
    reject('出错了');
  }
};

Promise.reject(thenable)
.catch(e => {
  console.log(e === thenable)
})
// true
````
## <a id='a9'>9.应用</a>
### 延时等待
````javascript
function wait(duration){
    return new Promise((resolve, reject)=>{
        setTimeout(resolve,duration);
    })
}
wait(1000).then(()=>{console.log('hello')}).then(()=>{return wait(2000)}).then(()=>{console.log('world')})
````
### 加载图片
````javascript
const preloadImage = function (path) {
  return new Promise(function (resolve, reject) {
    var image = new Image();
    image.onload  = resolve;
    image.onerror = reject;
    image.src = path;
  });
};
````
## <a id='a10'>10.参考资料</a>
* [ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/promise)
* [JavaScript Promise启示录](https://segmentfault.com/a/1190000000492290)
* [一道关于Promise应用的面试题](http://www.cnblogs.com/dojo-lzz/p/5495671.html)
