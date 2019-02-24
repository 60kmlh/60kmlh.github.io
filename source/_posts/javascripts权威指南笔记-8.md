---
title: 《javascript权威指南》第八章读书笔记
date: 2017-03-18
tags: ['function']
categories: ['笔记']
---
## 8.1函数定义
(1) 函数定义表达式

声明的变量会提升，赋值操作不会提升。
````javascript
var fn = function(){
    //do somethings
}
````
(2) 函数声明语句

函数声明语句会被提升到作用域顶部。
````javascript
function fn(){
    //do somethings
}
````
## 8.2函数调用
4种方式调用函数：

1. 作为函数

    ECMAScript 3和ECMAScript 5的非严格模式下规定，函数调用的调用上下文是全局对象。严格模式下，上下文对象是undefined。
    ````javascript
    'use strict';
    function fn(){
        console.log(this);
    }
    fn()//undefined
    ````
2. 作为方法

    将函数保存在一个对象的属性里，作为该对象的方法进行调用。

    任何函数作为方法调用都会传入一个隐式的实参，即调用该方法的对象。
    ````javascript
    var a=1;
    function fn1(){
        console.log(this.a);
    }
    var o={
        a:2,
        m:function(){
            console.log(this.a);
        }
    }
    console.log(fn1())//1
    console.log(o.m())//2
    ````
3. 作为构造函数

    通过new关键字进行构造函数调用。构造函数调用创建一个新的空对象，这个对象继承构造函数的prototype属性，构造函数将这个新对象用作其调用上下文，因此构造函数可以使用this关键字引用这个新对象。

    不使用return关键字时，返回该新对象。


4. 使用它们的call()或apply()方法间接调用

    call()方法使用自有的实参列表作为函数的实参。

    apply()则要求以数组的方式传入参数。

## 8.3函数的实参和形参
javascript函数调用不检查传入形参的个数。

当调用函数时传入的实参个数少于函数声明是指定的形参个数时，剩下的形参将设置为undefined 。

为了保持函数的适应性，应当给忽略的实参设置默认值。可使用||运算符。
````javascript
function fn(a){
    a=a||[]
}
````
当传入的实参个数大于指定的形参个数时，没有办法直接获得未命名值的引用。标示符arguments是指向实参对象的引用，可通过数字下标获得传入的实参值。
````javascript
function Max(){
    var max = Number.NEGATIVE_INFINITY;
    for(let i=0;i<arguments.length;i++){
        max=arguments[i]>max?arguments[i]:max;
    }
    return max
}
````
非严格模式下，arguments.callee指代当前正在执行的函数。arguments.caller指代调用当前正在执行的函数的函数。

匿名函数中可通过arguments.callee调用自身实现递归。
````javascript
var factorial = function(x){
    if (x<1) return 1;
    return x*arguments.callee(x-1)
}
````
javascript方法的形参并未进行类型检查，应当添加实参类型检查逻辑。
## 8.3作为值的函数
````javascript
var operators = {
    add:function(x,y){return x+y},
    subtract:function(x,y){return x-y},
    multiply:function(x,y){return x*y},
    divide:function(x,y){return x/y}
}

function operate(operation,operand1,operand2){
    if(typeof operators[operation] === 'function'){
        return operators[operation](operand1,operand2)
    }else{
        throw 'unkown operator'
    }
}
var j = operate('add', 'hello', operate('add',' ','world'))
console.log(j)//hello world
````
自定义函数属性
````javascript
function factorial(n){
    if(isFinite(n)&&n>0&&n==Math.round(n)){
        if(!(n in factorial)){
            factorial[n]=n*factorial(n-1)
        }
        return factorial[n]
    }else{
        return NaN
    }
}
factorial[1]=1;
console.log(factorial(4))//24
````
## 8.5作为命名空间的函数
````javascript
var extend = (function(){
    for(var i in {toString:null}){
        return function extend(o){
            for (var i=1;i<arguments.length;i++){
                var source = arguments[i];
                for(var prop in source){
                    o[prop] = source[prop]
                }
            }
        }
    }
    return function patched_extend(o){
        for (var i=1;i<arguments.length;i++){
            var source = arguments[i];
            for(var prop in source){
                o[prop] = source[prop]
            }
            for(var j=0;j<protoprops.length;j++){
                prop=protoprops[j];
                if(source.hasOwnProperty(prop)){
                    o[prop] = source[prop]
                }
            }
        }
    }
    var proptypes = ['toString','valueOf','constructor','hasOwnProperty']
}())
````
## 8.6闭包
javascript采用词法作用域，函数的执行依赖变量作用域，该作用域是在函数定义时决定的。函数定义时的作用域链，在函数执行时依然有效。

为实现这种词法作用域，javascript函数对象的内部状态不仅包含函数的代码逻辑，还必须引用当前的作用域链。

函数对象通过作用域链互相关联起来，函数体内部的变量都可以保存在函数作用域内，这种特性称为闭包。
````javascript
var scope = 'global scope';
function checkScope(){
    var scope = 'local scope';
    function f(){return scope}
    return f()
}
checkScope()//'local scope'
````
````javascript
var scope = 'global scope';
function checkScope(){
    var scope = 'local scope';
    function f(){return scope}
    return f
}
checkScope()()//'local scope'
````
利用闭包实现私有存取器方法
````javascript
function addPrivateProperty(o,name,predicate){
    var value;
    o['get'+name]=function(){return value};
    o['set'+name]=function(v){
        if(predicate&&!predicate(v)){
            throw Error('set'+name+':invalid value'+v)
        }else{
            value=v
        }
    }
}
var o = {};
addPrivateProperty(o,'Name',function(x){return typeof x==='string'})
o.setName('Frank');
console.log(o.getName());//'Frank'
o.setName(o) //Error: setName:invalid value[object Object]
````
this是javascript关键字，而不是变量，如果闭包在外部函数里无法访问this，除非外部函数将this转存为一个变量。arguments同理。
## 8.7函数属性、方法和构造函数
1. length属性
    在函数体里，arguments.length指向传入函数的实参长度，而函数的length属性指函数定义时给出的参数个数，即“形参”。
    ````javascript
    function check(args){
        var actual = args.length;
        var expected = args.callee.length;
        if(actual!==expected){
            throw Error('Expected'+expected+'args;got'+actual)
        }
    }
    function f(x,y,z){
        check(arguments);
        return x+y+z
    }
    ````
2. prototype
    当函数用作构造函数的时候，新创建的对象会从原型对象上继承属性。
3. call()和apply()方法
    call()和apply()的第一个参数是要调用函数的母对象，它是调用的上下文，函数体内通过this获得对它引用。

    ECMAScript 5的严格模式中，第一个实参会变成this的值。
    ````javascript
    'use strict'
    var a=1;
    var x={a:2}
    function f(){console.log(a)}
    f.call(x)//1
    f.aplly(x)//1
    ````
    ECMAScript 3和非严格模式中，传入null和undefined会被全局对象所替代。

    call()方法第一个调用上下文实参之后的所有实参就是要传入待调用函数的值。
    ````javascript
    f.call(o,1,2)
    ````
    apply()调用上下文实参之后的所有实参d都放入一个数组中。
    ````javascript
    f.apply(o,[1,2])
    ````
4. bind()方法
    ECMAScript 5新增bind()方法，用来将函数绑定至某个对象。

    在函数f()调用bind方法并传入一个对象o作为参数，将返回一个新的函数。调用这个新的函数，会把原始函数f()当做对象o的方法来调用。
    ````javascript
    function f(y){ return this.x+y }
    var o = {x:1};
    var g = f.bind(o);
    g(2)//3
    ````
    除了第一个实参外，传入bind()的参数也会绑定至this。

    bind()方法所返回的函数的length（形参数量）等于原函数的形参数量减去传入bind()方法中的实参数量（第一个参数以后的所有参数），因为传入bind中的实参都会绑定到原函数的形参
    ````javascript
    var sum = function(x,y){return x+y}//length为2
    var succ = sum.bind(null,1)//length为1
    succ(2)//3

    //var sum = function(x,y){return x+y}//length为2
    //var succ = sum.bind(null,1,2)//length为0
    //succ()//3
    ````
    当bind()所返回的函数用作构造函数的时候， 传入bind()的this将被忽略，实参会全部传入原函数。

    bind()方法返回的构造函数不包含prototype属性，将这些绑定的函数用作构造函数所创建的对象会从原始的未绑定的构造函数中继承prototype。
    ````javascript
    function original(x){
        this.a = 1;
        this.b = function(){return this.a + x}
    }
    var obj={
        a = 10
    }
    var newObj = new(original.bind(obj, 2)); //传入了一个实参2
    console.log(newObj.a);  //输出1, 说明返回的函数用作构造函数时obj(this的值)被忽略了
    console.log(newObj.b()); //输出3 ，说明传入的实参2传入了原函数original
    ````

    ECMAScript 3版本的bind方法()
    ````javascript
    if(!Function.prototype.bind){
        Function.prototype.bind(o,/*,arguments*/){
            var self = this, boundArgs = arguments;
            return function(){
                var args = [], i;
                for(i=1;i<boundArgs.length;i++){args.push(bonudArgs[i])}
                for(i=0;i<arguments.length;i++){args.push(arguments[i])}
                return self.apply(o,args)
            }
        }
    }
    ````
5. toString()方法
    大多数函数的toString()方法的实现都返回函数的完整源码。
    
    内置函数往往返回一个“[native code]”的字符串作为函数体。
6. Function()构造函数
    函数可通过Function()构造函数来定义。
    ````javascript
    var f = new Function('x','y','return x+y')
    ````
    Function()构造函数允许javascript运行时动态创建并编译函数。

    每次调用Function()构造函数都回解析函数体，并创建新的函数对象。在循环中执行，会影响执行效率。

    Function()构造函数创建的函数不使用词法作用域，函数体代码的编译综会在顶层函数执行。
    ````javascript
    var scope = 'global';
    function constructFunciton(){
        var scope = 'local';
        return new Function('return scope')
    }
    constructFunciton()()//'global'
    ````
7. 可调用对象
    截至目前为止，两个可调用对象在javascript中的实现不能算作函数。

    ie8机之前的版本的客户端方法使用了可调用的宿主对象，而不是内置的函数对象。

    另一个常见的可调用对象是RegExp对象，非javascript的标准特性。

    检查对象是否为真正的函数对象。
    ````javascript
    function isFunction(x){
        return Object.prototype.toString.call(x)==='[object Function]'
    }
    isFunction(window.alert)//IE8下为false
    ````
## 8.8函数式编程
1. 使用函数处理数组
    自定义map()函数
    ````javascript
    var map = Array.prototype.map?function(a,f){return a.map(f)}:function(a,f){
        var results = [];
        for(var i = 0,len = a.length;i<len;i++;){
            if(i in a){
                results[i] = f.call(null,a[i],i,a)
            }
        }
        return results
    }
    自定义reduce()函数
    ````javascript
    var reduce = Array.prototype.reduce?function(a,f,initial){
        if(arguments.length>2){
            return a.reduce(f,initial)
        }else{return a.reduce(f)}
    }:function(a,f,initial){
        var i = 0,len = a.length,accumulator;
        if(arguments.length>2){accumulator=initial}
        else{
            if(len == 0){throw TypeError()}
            while(i < len){
                if(i in a){
                    accumulator=a[i++];
                    break;
                }else{
                    i++
                }
            }
            if(i == len){throw TypeError()}
        }
        while(i < len){
            if(i in a){
                accumulator=f.call(undefined,accumulator,a[i],i,a);
                i++
            }
        }
        return accumulator
    }
    ````
2. 高阶函数
    高阶函数(higher-order function)即操作函数的函数，接收一个或者多个函数作为参数，返回一个新的函数。
    ````javascript
    function not(f){
        return function(){
            var result = f.apply(this,arguments)
            return !return
        }
    }
    function even(){
        return x%2 === 0
    }
    var odd = not(even);
    [1,3,5,7,9].every(odd)//true
    ````
3. 不完全函数
    在JavaScript中，不完全函数是一种函数变换技巧，即把一次完整的函数调用拆成多次函数调用，每次传入的实参都是完整实参的一部分，每个拆分开的函数叫做不完全函数，每次函数调用叫做不完全调用，这种变换的特点是每次调用都返回一个函数，直到得到最终运行结果为止。
    
    举一个简单的例子，将对函数f(1,2,3,4,5)的调用修改为等价的f(1,2)(3,4)(5,6)，后者包含三次调用，和每次调用相关的函数就是“不完全函数”。
    ````javascript
    //实现一个工具函数，将类数组（或对象）转换为真正的数组
    //在后面的示例代码中用到了这个方法将arguments对象转换为真正的数组
    function array(a,n){
        return Array.prototype.slice.call(a,n||0);
    }

    //这个函数的实参传递至左侧
    function partialLeft(f){
        var args = arguments;   //保存外部的实参数组
        return function(){      //并返回这个函数
            var a = array(args,1);  //从第一个元素开始处理args
            a = a.concat(array(arguments)); //然后增加所有的内部实参
            return f.apply(this,a); //然后基于这个实参列表调用f()
        }
    }

    //这个函数的实参传递至右侧
    function partialRight(f){
        var args = arguments;   //保存外部的实参数组
        return function(){      //并返回这个函数
            var a = array(arguments);   //从内部参数开始
            a = a.concat(array(args,1));    //从第一个元素开始处理args
            return f.apply(this,a); //然后基于这个实参列表调用f()
        }
    }

    //这个函数的实参传递至左侧
    //如果参数为undefined,用后面的实参填充undefined
    function partial(f){
        var args = arguments;   //保存外部实参数组
        return function(){
            var a = array(args,1);  //从外部args开始
            //遍历args，从内部实参填充undefined值
            for(var i=0,j=0;i<a.length;i++){
                if (a[i]===undefined) a[i] = arguments[j++];
            }
            //现在将剩下的内部实参都追加进去
            a = a.concat(array(arguments,j));
            return f.apply(this,a);
        };
    }

    //这个函数带有三个实参
    var f = function(x,y,z){return x*(y-z);};

    //注意这三个不完全调用之间的区别
    console.log(partialLeft(f,2)(3,4)); //-2，绑定第一个实参：2*(3-4)
    console.log(partialRight(f,2)(3,4));    //6，绑定第一个实参：3*(4-2)
    console.log(partial(f,2)(3,4));     //-2，绑定第一个实参：2*(3-4)
    console.log(partial(f,undefined,2)(3,4));   //-6，绑定第一个实参：3*(2-4)
    ````
4. 记忆
    在函数式编程中，将上次计算的结果缓存起来，这种缓存技巧叫“记忆”(memorization)。
    ````javascript
    //返回f()带有记忆功能的版本
    function memorize(f){
        var cache = {};
        return function(){
            //将实参转换为字符串形式，并将其用作缓存的键。
            var key = arguments.length+Array.prototype.join.call(arguments,',');
            if(key in cache){return cache[key]}
            else{return cache[key] = f.apply(this,arguments)}
        }
    }
    ````
