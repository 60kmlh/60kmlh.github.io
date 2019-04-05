---
title: 浏览器的Event Loop机制
date: 2018-07-21
tags: ['浏览器', 'eventloop']
categories: ['浏览器']
---
## 单线程的JavaScript

> JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？
>
> 所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。
<!--more-->
## 任务队列 (Task Queue)

>单线程就意味着，所有任务需要排队，前一个任务结束，才会执行后一个任务。如果前一个任务耗时很长，后一个任务就不得不一直等着。
>
>所有任务可以分成两种，一种是同步任务（synchronous），另一种是异步任务（asynchronous）。同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；异步任务指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

## Event Loop机制

### 执行栈与事件队列

当我们调用一个方法的时候，js会生成一个与这个方法对应的执行环境（context），又叫执行上下文。这个执行环境中存在着这个方法的私有作用域，上层作用域的指向，方法的参数，这个作用域中定义的变量以及这个作用域的this对象。 而当一系列方法被依次调用的时候，因为js是单线程的，同一时间只能执行一个方法，于是这些方法被排队在一个单独的地方。这个地方被称为执行栈。

当一个脚本第一次执行的时候，js引擎会解析这段代码，并将其中的同步代码按照执行顺序加入执行栈中，然后从头开始执行。如果当前执行的是一个方法，那么js会向执行栈中添加这个方法的执行环境，然后进入这个执行环境继续执行其中的代码。当这个执行环境中的代码 执行完毕并返回结果后，js会退出这个执行环境并把这个执行环境销毁，回到上一个方法的执行环境。这个过程反复进行，直到执行栈中的代码全部执行完毕。
````javascript
function foo(b) {
  var a = 10;
  return a + b + 11;
}

function bar(x) {
  var y = 3;
  return foo(x * y);
}

console.log(bar(7)); // 返回 42
````

函数调用形成了一个栈帧。

当调用 bar 时，创建了第一个帧 ，帧中包含了 bar 的参数和局部变量。当 bar 调用 foo 时，第二个帧就被创建，并被压到第一个帧之上，帧中包含了 foo 的参数和局部变量。当 foo 返回时，最上层的帧就被弹出栈（剩下 bar 函数的调用帧 ）。当 bar 返回的时候，栈就空了。

js引擎遇到一个异步事件后并不会一直等待其返回结果，而是会将这个事件挂起，继续执行执行栈中的其他任务。当一个异步事件返回结果后，js会将这个事件加入与当前执行栈不同的另一个队列，我们称之为事件队列。被放入事件队列不会立刻执行其回调，而是等待当前执行栈中的所有任务都执行完毕， 主线程处于闲置状态时，主线程会去查找事件队列是否有任务。如果有，那么主线程会从中取出排在第一位的事件，并把这个事件对应的回调放入执行栈中，然后执行其中的同步代码...，如此反复，这样就形成了一个无限的循环。这就是这个过程被称为“事件循环（Event Loop）”的原因。
### 同步任务和异步任务

Javascript单线程任务被分为同步任务和异步任务，同步任务会在调用栈中按照顺序等待主线程依次执行，异步任务会在异步任务有了结果后，将注册的回调函数放入任务队列中等待主线程空闲的时候（调用栈被清空），被读取到栈内等待主线程的执行。

### 事件循环

不同的异步任务被分为两类：微任务（micro task）和宏任务（macro task）。

以下事件属于宏任务：
* script全部代码、setTimeout、setInterval、setImmediate（浏览器暂时不支持，只有IE10支持，具体可见MDN）、I/O、UI Rendering。
以下事件属于微任务：
* Process.nextTick（Node独有）、Promise、Object.observe(废弃)、MutationObserver。


**事件循环的进程模型**

* 选择当前要执行的任务队列，选择任务队列中最先进入的任务，如果任务队列为空即null，则执行跳转到微任务（MicroTask）的执行步骤。
* 将事件循环中的任务设置为已选择任务。
* 执行任务。
* 将事件循环中当前运行任务设置为null。
* 将已经运行完成的任务从任务队列中删除。
* microtasks步骤：进入microtask检查点。
* 更新界面渲染。
* 返回第一步。

执行栈在执行完同步任务后，查看执行栈是否为空，如果执行栈为空，就会去检查微任务(microTask)队列是否为空，如果为空的话，就执行Task（宏任务），否则就一次性执行完所有微任务。

每次单个宏任务执行完毕后，检查微任务(microTask)队列是否为空，如果不为空的话，会按照先入先出的规则全部执行完微任务(microTask)后，设置微任务(microTask)队列为null，然后再执行宏任务，如此循环。

**总结：当当前执行栈执行完毕时会立刻先处理所有微任务队列中的事件，然后再去宏任务队列中取出一个事件。同一次事件循环中，微任务永远在宏任务之前执行。**
## 例子

````javascript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});
console.log('script end');
````
执行分析：
1. 执行同步代码，将宏任务（Tasks）和微任务(Microtasks)划分到各自队列中。
2. 执行宏任务后，检测到微任务(Microtasks)队列中不为空，执行Promise1，执行完成Promise1后，调用Promise2.then，放入微任务(Microtasks)队列中，再执行Promise2.then。
3. 当微任务(Microtasks)队列中为空时，执行宏任务（Tasks），执行setTimeout callback，打印日志。
4. 清空Tasks队列和JS stack。

|事件顺序|宏任务|微任务|执行栈|输出
|---|---|---|---|---|
|第一次|run script、 setTimeout callback|Promise then|script|script start、script end|
|第二次|run script、 setTimeout callback|Promise2 then|Promise2 callback|start、script end、promise1|
|第三次|etTimeout callback|空|setTimeout callback|script start、script end、promise1、promise2|
|第四次|etTimeout callback|空|空|script start、script end、promise1、promise2、setTimeout|

执行帧动画：[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

### setTimeout

函数 setTimeout 接受两个参数：待加入队列的消息和一个延迟（可选，默认为 0）。这个延迟代表了消息被实际加入到队列的最小延迟时间。如果队列中没有其它消息，在这段延迟时间过去之后，消息会被马上处理。但是，如果有其它消息，setTimeout 消息必须等待其它消息处理完。因此第二个参数仅仅表示最少延迟时间，而非确切的等待时间。

````javaScript
const s = new Date().getSeconds();

setTimeout(function() {
  // 输出 "2"，表示回调函数并没有在 500 毫秒之后立即执行
  console.log("Run after " + (new Date().getSeconds() - s) + " seconds");
}, 500);

while(true) {
  if(new Date().getSeconds() - s >= 2) {
    console.log("Good, looped for 2 seconds");
    break;
  }
}
//输出：Good, looped for 2 seconds
//      Run after 2 seconds
````

> HTML5标准规定了setTimeout()的第二个参数的最小值（最短间隔），不得低于4毫秒，如果低于这个值，就会自动增加。在此之前，老版本的浏览器都将最短间隔设为10毫秒。另外，对于那些DOM的变动（尤其是涉及页面重新渲染的部分），通常不会立即执行，而是每16毫秒执行一次。这时使用requestAnimationFrame()的效果要好于setTimeout()。

再看看这段代码
````javaScript
setTimeout(() => {
	console.log(2)
}, 2)

setTimeout(() => {
	console.log(1)
}, 1)

setTimeout(() => {
	console.log(0)
}, 0)
````

按照上面的规范，低于4毫秒的自动变为4毫秒，因此三个setTimeout进入异步队列的顺序的依次从上到下的，即打印出 2 1 0 。

然而在FireFox 64.0.2 (32 位) 环境下，setTimeout的时间是可以设置小于4毫秒的，因此输出是 0 1 2 。

在Chrome 72.0.3626.121(正式版本)(32 位) 环境下，打印的是 1 0 2 。

从测试结果可以看出，0ms和1ms的延时效果是一致的。原因是Blink内核对于传入时间有这样一行判断：

````c++
// https://chromium.googlesource.com/chromium/blink/+/master/Source/core/frame/DOMTimer.cpp#93

double intervalMilliseconds = std::max(oneMillisecond, interval * oneMillisecond); 
````
**可以看出传入0和传入1结果都是oneMillisecond，即1ms。**
### setInterval

对于setInterval(fn,ms)来说，我们已经知道不是每过ms秒会执行一次fn，而是每过ms秒，会有fn进入Event Queue。一旦setInterval的回调函数fn执行时间超过了延迟时间ms，那么就完全看不出来有时间间隔了。

````javaScript
const s = new Date().getSeconds();

setInterval(function() {
  let _s = new Date().getSeconds();
  while(true) {
    if(new Date().getSeconds() - _s >= 2) {
      break;
    }
  }
  console.log("Run after " + (new Date().getSeconds() - s) + " seconds");
}, 500);
//每隔2秒打印一次，而不是500毫秒
````

### async/await

async/await 在底层转换成了 promise 和 then 回调函数。也就是说，这是 promise 的语法糖。

每次我们使用 await, 解释器都创建一个 promise 对象，然后把剩下的 async 函数中的操作放到 then 回调函数中。

然而对于下面这段代码，不同的浏览器，执行结果是不一致的。
````javaScript
async function async1() {
  console.log("a");
  await async2();
  console.log("b");
}
async function async2() {
  console.log('c');
}
async1();
new Promise(function (resolve) {
  console.log("d");
  resolve();
}).then(function () {
  console.log("e");
});
````
而在 FireFox 64.0.2 (32 位) 环境下，输出 a c d e b 。

在 Chrome 72.0.3626.121(正式版本)(32 位) 环境下，输出 a c d b e 。

原因是由于ECMAScript规范的更新（ [ TC39 新决议](https://github.com/tc39/ecma262/pull/1250) ），决定await将直接使用Promise.resolve()相同语义，导致不同版本的浏览器对于async的实现是不一样的。

在旧的规范中，await p 近似等于 return new Promise(resolve => resolve(p))。

在新的规范中，await p 近似等于 return Promise.resolve(p)。

如果传递给 await 的值已经是一个 Promise，则返回Promise.resolve(p)，那么这种优化避免了再次创建 Promise 包装器。

> 如果 RESOLVE(p) 严格按照标准，应该是产生一个新的 promise，尽管该 promise 确定会 resolve 为 p，但这个过程本身是异步的，也就是现在进入 job 队列的是新 promise 的 resolve 过程，所以该 promise 的 then 不会被立即调用，而要等到当前 job 队列执行到前述 resolve 过程才会被调用，然后其回调（也就是继续 await 之后的语句）才加入 job 队列，所以时序上就晚了。

对于以上代码，在不同规范下转化为Promise之后如下：

**旧规范：**
````javaScript
async function async1 () {
  console.log('a')
  new Promise(resolve => {
    resolve(async2())
  }).then(res => {
    console.log('b')
  })
}
async function async2 () {
  console.log('c')
}
async1()
new Promise(function (resolve) {
  console.log('d')
  resolve()
}).then(function () {
  console.log('e')
})
````
因为 async2() 返回 promise, 所以进一步转化就是：
````javaScript
async function async1 () {
  console.log('a')
  new Promise(resolve => {
    Promise.resolve().then(() => {
      async2().then(resolve)
    })
  }).then(res => {
    console.log('b')
  })
}
async function async2 () {
  console.log('c')
}
async1()
new Promise(function (resolve) {
  console.log('d')
  resolve()
}).then(function () {
  console.log('e')
})
````
输出：a c d e b

**新规范：**
````javaScript
async function async1 () {
  console.log('a')
  Promise.resolve(async2()).then(res => {
    console.log('b')
  })
}
async function async2 () {
  console.log('c')
}
async1()
new Promise(function (resolve) {
  console.log('d')
  resolve()
}).then(function () {
  console.log('e')
})
````
输出：a c d b e

## 参考资料
* [并发模型与事件循环](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)
* [JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
* [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
* [一次弄懂Event Loop](https://juejin.im/post/5c3d8956e51d4511dc72c200)
* [async/await 在chrome 环境和 node 环境的 执行结果不一致，求解？](https://www.zhihu.com/question/268007969)
* [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)
* [Event Loop的规范和实现](https://zhuanlan.zhihu.com/p/33087629)
* [window.setTimeout](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setTimeout)