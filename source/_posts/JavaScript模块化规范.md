---
title: JavaScript模块化规范
date: 2018-01-06
tags: ['模块化', 'js']
categories: ['规范']
---
## 模块化的进化过程
* 全局function模式：将不同的功能封装成不同的全局函数

  缺点：污染全局命名空间，容易引起命名冲突或数据不安全，而且模块成员之间看不出直接关系。
<!--more-->
````javascript
function m1(){
  //...
}
function m2(){
  //...
}
````
* namespace模式：简单对象封装

  缺点：数据不安全(外部可以直接修改模块内部的数据)、
````javascript
let myModule = {
  data: 'original data',
  print() {
    console.log(`data: ${this.data}`)
  }
}
myModule.data = 'changed data' //能直接修改模块内部的数据
myModule.foo() // data: changed data
````
* IIFE模式：匿名函数自调用(闭包)

  将数据和行为封装到一个函数内部，通过给window添加属性来向外暴露接口，数据是私有的，外部只能通过暴露的方法操作。

  缺点：无法解决模块间依赖关系的问题。
````javascript
// module.js文件
(function(window) {
  let data = 'original data'
  
  function print() {
    console.log(`data: ${data}`)
    privateFn()
  }

  function privateFn() {
    //内部私有的函数
    console.log('privateFn')
  }
  //往window暴露模块
  window.myModule = { print }
})(window)
````
````html
<!-- index.html文件  -->
<script type="text/javascript" src="module.js"></script>
<script type="text/javascript">
    myModule.print()
    console.log(myModule.data) //undefined 不能访问模块内部数据
    myModule.data = 'changed data' //不是修改的模块内部的data
    myModule.print() //original data 没有改变
</script>
````
* IIFE模式增强：引入依赖
 
  解决了模块间依赖关系的问题
````javascript
// module.js文件
(function(window, $) {
  let data = 'original data'
  
  function print() {
    console.log(`data：${data}`)
    $('body').css('background', 'red')
  }
  //往window暴露模块
  window.myModule = { foo, bar }
})(window, jQuery)
````
````html
  <!-- index.html文件  -->
  <!-- 引入的js必须有一定顺序 -->
  <script type="text/javascript" src="jquery-1.10.1.js"></script>
  <script type="text/javascript" src="module.js"></script>
  <script type="text/javascript">
    myModule.print()
  </script>
````
必须先引入jQuery库，就把这个库当作参数传入。这样做除了保证模块的独立性，还使得模块之间的依赖关系变得明显。

引入多个&lt;script&gt;后出现出现问题：

* 请求过多

  首先我们要依赖多个模块，那样就会发送多个请求，导致请求过多

* 依赖模糊

  我们不知道他们的具体依赖关系是什么，也就是说很容易因为不了解他们之间的依赖关系导致加载先后顺序出错。

* 难以维护

  以上两种原因就导致了很难维护，很可能出现牵一发而动全身的情况导致项目出现严重的问题。
## 模块化规范
### CommonJS

Node 应用由模块组成，采用 CommonJS 模块规范。每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。在服务器端，模块的加载是运行时同步加载的；在浏览器端，模块需要提前编译打包处理。

1. 基本语法

* 定义暴露模块：

  module.exports = value

  为了方便，Node为每个模块提供一个exports变量，指向
  module.exports。因此也可以使用exports.xxx = value

  注意，不能直接将exports变量指向一个值，因为这样等于切断了exports与module.exports的联系。

* 引入模块：

  require(xxx),如果是第三方模块，xxx为模块名；如果是自定义模块，xxx为模块文件路径

2. 加载机制

CommonJS模块的加载机制是，输入的是被输出的值的拷贝。也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。
````javascript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
````
````javascript
// main.js
var counter = require('./lib').counter;
var incCounter = require('./lib').incCounter;

console.log(counter);  // 3
incCounter();
console.log(counter); // 3

````
上面代码说明，counter输出以后，lib.js模块内部的变化就影响不到counter了

3.require()的内部处理流程

require命令是CommonJS规范之中，用来加载其他模块的命令。它其实不是一个全局命令，而是指向当前模块的module.require命令，而后者又调用Node的内部命令Module._load。

````javascript
Module._load = function(request, parent, isMain) {
  // 1. 检查 Module._cache，是否缓存之中有指定模块
  // 2. 如果缓存之中没有，就创建一个新的Module实例
  // 3. 将它保存到缓存
  // 4. 使用 module.load() 加载指定的模块文件，
  //    读取文件内容之后，使用 module.compile() 执行文件代码
  // 5. 如果加载/解析过程报错，就从缓存删除该模块
  // 6. 返回该模块的 module.exports
};
````
上面的第4步，采用module.compile()执行指定模块的脚本，逻辑如下。
````javascript
Module.prototype._compile = function(content, filename) {
  // 1. 生成一个require函数，指向module.require
  // 2. 加载其他辅助方法到require
  // 3. 将文件内容放到一个函数之中，该函数可调用 require
  // 4. 执行该函数
};
````
### AMD

AMD是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义"。CommonJS规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。AMD规范则是非同步加载模块，允许指定回调函数。由于Node.js主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以CommonJS规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用AMD规范。

RequireJS是AMD规范的一种实现。

1. 基本语法

* 定义暴露模块:
如果被定义的模块是一个独立模块，不需要依赖任何其他模块，可以直接用define方法生成。

````javascript
define({
    method1: function() {},
    method2: function() {},
});
````
另一种等价的写法是，把对象写成一个函数，该函数的返回值就是输出的模块。
````javascript
define(function () {
	return {
	    method1: function() {},
		  method2: function() {},
    };
});
````
如果被定义的模块需要依赖其他模块，则define方法必须采用下面的格式。

define方法的第二个参数是一个函数，当前面数组的所有成员加载成功后，它将被调用。它的参数与数组的成员一一对应。
````javascript
define(['module1', 'module2'], function(m1, m2) {
    return {
        method: function() {
          m1.methodA();
			    m2.methodB();
        }
    };
});
````
如果依赖的模块很多，参数与模块一一对应的写法非常麻烦。

为了避免像上面代码那样繁琐的写法，RequireJS提供一种更简单的写法。
````javascript
define(
    function (require) {
        var dep1 = require('dep1'),
            dep2 = require('dep2'),
            dep3 = require('dep3'),
            dep4 = require('dep4'),
            dep5 = require('dep5'),
            dep6 = require('dep6'),
            dep7 = require('dep7'),
            dep8 = require('dep8');
            ...
    }
});
````
* 引入模块
````javascript
require(['foo', 'bar'], function ( foo, bar ) {
        foo.doSomething();
});
````
require方法也可以用在define方法内部。
````javascript
define(function (require) {
   var otherModule = require('otherModule');
});
````
### CMD

CMD 是 [sea.js](https://github.com/seajs/seajs) 在推广过程中对模块定义的规范化产出。

CMD规范专门用于浏览器端，模块的加载是异步的，模块使用时才会加载执行。CMD规范整合了CommonJS和AMD规范的特点。
1. 基本语法
* 暴露模块：
````javascript
//定义没有依赖的模块
define(function(require, exports, module){
  exports.xxx = value
  module.exports = value
})

//定义有依赖的模块
define(function(require, exports, module){
  //引入依赖模块(同步)
  var module2 = require('./module2')
  //引入依赖模块(异步)
    require.async('./module3', function (m3) {
    })
  //暴露模块
  exports.xxx = value
})
````
* 引入模块
````javascript
define(function (require) {
  var m1 = require('./module1')
  var m4 = require('./module4')
  m1.show()
  m4.show()
})
````
2. AMD 与 CMD 的区别

* 对于依赖的模块，AMD 是提前执行，CMD 是延迟执行。不过 RequireJS 从2.0开始，也改成了可以延迟执行（根据写法不同，处理方式不同）。CMD 推崇 as lazy as possible.

* CMD 推崇依赖就近，AMD 推崇依赖前置
### ES6模块化规范

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

1. 基本语法
export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。
````javascript
/** 定义模块 math.js **/
var basicNum = 0;
var add = function (a, b) {
    return a + b;
};
export { basicNum, add };
/** 引用模块 **/
import { basicNum, add } from './math';
function test(ele) {
    ele.textContent = add(99 + basicNum);
}
````
2. import()

import命令会被 JavaScript 引擎静态分析，先于模块内的其他语句执行（import命令叫做“连接” binding 其实更合适）。所以，下面的代码会报错。
````javascript
// 报错
if (x === 2) {
  import MyModual from './myModual';
}
````
上面代码中，引擎处理import语句是在编译时，这时不会去分析或执行if语句，所以import语句放在if代码块之中毫无意义，因此会报句法错误，而不是执行时错误。也就是说，import和export命令只能在模块的顶层，不能在代码块之中（比如，在if代码块之中，或在函数之中）。

因此，有一个提案，建议引入import()函数，完成动态加载。import()返回一个 Promise 对象。
````javascript
button.addEventListener('click', event => {
  import('./dialogBox.js')
  .then(dialogBox => {
    dialogBox.open();
  })
  .catch(error => {
    /* Error handling */
  })
});
````
3. ES6 模块与 CommonJS 模块的差异

* CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。

* CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

## 参考资料

* [前端模块化详解](https://juejin.im/post/5c17ad756fb9a049ff4e0a62)
* [CommonJS规范](http://javascript.ruanyifeng.com/nodejs/module.html)
* [RequireJS和AMD规范](http://javascript.ruanyifeng.com/tool/requirejs.html)
* [ES6入门-Module的语法](http://es6.ruanyifeng.com/#docs/module)