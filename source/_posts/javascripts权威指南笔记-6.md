---
title: 《javascript权威指南》第六章读书笔记
date: 2017-03-11
tags: ['object']
categories: ['笔记']
---
## 6.1 创建对象

### 创建对象三种的方式

1. 使用对象直接量

创建对象最简单的方式就是在JavaScript代码中使用对象直接量(key / value)。

在ECMAScript 5中，对象直接量中的最后一个属性后的逗号将忽略，且在ECMAScript 3的大部分实行中也可以忽略这个逗号，但在IE中则报错。

所有通过对象直接量创建的对象具有同一个原型对象，即Object.prototype。Object.prototype没有原型。

2. 使用new运算符

关键字new后跟随一个构造函数使用，创建的对象的原型是构造函数的prototype属性的值。

所有通过对象直接量创建的对象具有同一个原型对象，即Object.prototype。Object.prototype没有原型。

3. 使用Object.create() 

ECMAScript 5中使用Object.create(obj)传入原型对象，创建的对象会继承该原型的属性。传入Null则创建一个没有原型的对象,不继承任何属性。

在ECMASCript 3中模拟原型继承
````javascript
function inherit(p) {
    if (p == null) throw TypeError();
    if (Object.create) return Object.create(p);
    var t = typeof p;
    if (t != 'function' && t != 'object') throw TypeError();
    function f() { };
    f.prototype = p;
    return new f()
````
## 6.2 属性的查询和设置
可以通过点(.)或者方括号([])运算符获取属性的值。

Javascript对象具有'自有属性'，也有原型属性是从原型对象继承而来的。

查询对象o的属性x，如果在o中找不到x，则在o的原型对象中查找属性x。如果该原型对象也没有属性x，但这个原型对象也有原型，那么继续在这个原型对象的原型上执行查询，直到找到x属性或者一个原型是null的对象为止。即对象的原型属性构成一个'链'。
````javascript
var o = {}
o.x = 1;
var p = inherit(o)
p.y = 2;
var q = inherit(p)
q.z = 3;
var s = q.toString()
console.log(s)//[object Object]
console.log(q.x+q.y)//3
````
如果允许属性赋值操作，那么它总是在原始对象上进行属性创建或者对已有属性进行赋值，而不会去修改原型链。

查询属性才和继承有关，设置属性则无关，改特性可以选择性得覆盖继承的属性。

查询一个不存在的属性不会报错，返回undefined。如果对象不存在，查找这个不存在的对象的属性就会报错。
````javascript
var len = book&&book.substitle&&book.substitle.length;
````
## 6.3删除属性
使用delete刪除屬性。delete只是断开属性与宿主之间的联系，而不会去操作属性中的属性。
````javascript
var a = {p:{x:1}};
var b = a.p;
delete a.p;
console.log(a.p);//undefined
console.log(b);//{x:1}
````
delete不行删除可configurable为false的属性。
## 6.4检测属性
检测某个属性是否存在于某个对象中。

1. in运算符

如果对象的自有属性或继承属性包含这个属性，则返回true。
```javascript
var a = {x:1};
console.log('x' in a);//true
console.log('toString' in a);//true
````

2. hasOwnProperty()

该方法用来检测给定的名字是否是对象的自有属性。
````javascript
var a = {x:1};
console.log(a.hasOwnProperty('x'));//true
console.log(a.hasOwnProperty('toString'));//false
````

3. propertyIsEnumerable()

检测到是自有属性且这个属性的enumerable特性为true是，返回true。
````javascript
var o = inherit({x:1});
o.y = 2;
console.log(o.propertyIsEnumerable('x'))//true
console.log(o.propertyIsEnumerable('y'))//false
console.log(o.propertyIsEnumerable('toString'))//false
````
## 6.5枚举属性
使用for/in便利对象所有可枚举的属性。
````javascript
var o = {x:1 ,y:2, z:3};
console.log(o.propertyIsEnumerable('toString'))//fasle
for(prop in o){
    console.log(prop)
}//x y z
````
ECMAScript 5中使用Object.keys(obj)返回一个数组，数组由对象中的可枚举的自有属性组成。
````javascript
var o = {x:1,y:2};
Object.defineProperty(o,'y',{enumerable:false})
console.log(Object.keys(o));//['x']
````
ECMAScript 5中使用Object.getOwnPropertyNames(obj)返回对象所有的自有属性。
````javascript
var o = {x:1,y:2};
Object.defineProperty(o,'y',{enumerable:false})
console.log(Object.getOwnPropertyNames(o));//['x','y']
````
## 6.6属性的getter和setter
在ECMAScript 5中 ，对象的属性可以有一个或两个函数代替，即getter和getter。由getter和setter定义的属性叫做'存取器属性'(accessor property)，不同于'数据属性'(data property)。

存取器属性定义为一个或两个和属性同名的函数，不使用function关键字，而是使用get和set。

存取器属性不具有可写性(writable attribute)。
````javascript
var serialNum={
    $n:0,
    get next(){
        return this.$n++
    },
    set next(n){
        if(n>this.$n) this.$n=n*3;
        else throw '序列号的值不能比当前值小'
    }
}
console.log(serialNum.next)//0
console.log(serialNum.next)//1
serialNum.next=10;
console.log(serialNum.next)//30
console.log(serialNum.next)//31
````
## 6.7属性的特性
数据属性的4个特性是：value writable enumerable configurable。

存取器属性不具有value和writable特性，其可写性有setter方法是否存在决定。

存取器属性的4个特性是：get set enumerable configurable。

通过调用Object.getOwnPropertyDescriptor(obj,propName)获得某个对象特定属性的属性描述符。
````javascript
console.log(Object.getOwnPropertyDescriptor({x:1},'x'))//{ value: 1, writable: true, enumerable: true, configurable: true }
````
设置属性的特性则是使用Object.defineProperty(obj,propName,desc)和Object.defineProperties(obj,desc)

赋值属性的特性
````javascript
Object.defineProperty(Object.prototype,'extend',{
    writable:true,
    enumerable:false,
    configurable:true,
    value:function(o){
        var names=Object.getOwnPropertyNames(o);
        for( let i=0;i<names.length;i++){
            if(names[i] in this) continue;
            var desc = Object.getOwnPropertyDescriptor(o,names[i]);
            Object.defineProperty(this,names[i],desc)
        }
    }
})
````
## 6.8对象的3个属性

1. 原型(prototype)

ECMAScript 5中使用Object.getProprttyOf(obj)查询对象的原型。

ECMAScript 3中使用o.constructor.prototype检查对象的原型(不可靠)。

使用isPrototypeOf检测一个对象是否是另一个对象的原型。

2. 类属性(class attribute)

查询方法
````javascript
function classOf(o){
    if(o===null) return 'Null';
    if(o===undefined) return 'Undefined';
    return Object.prototype.toString.call(o).slice(8,-1)
}
````
3. 可扩展性(extensible attribute)

对象的可扩展性表示是否可以给对象添加新属性。

所以内置对象和自定义对象都是显式可扩展的。

可以通过Object.esExtensible(obj)来检测对象是否是可扩展的

通过Object.preventExtensions(obj)、Object.seal(obj)、Object.freeze(obj)来将对象设置为不可扩展的

## 6.9序列化对象
对象序列化(serialization)是指对象的状态和字符串的互相转换。

ECMAScript 5内置函数JSON.stringify()和JSON.parse()来序列化和还原对象。
````javascript
var o = {x:1,y:{a:1}};
var s = JSON.stringify(o);
console.log(s);//{"x":1,"y":{"a":1}}
console.log(JSON.parse(s));//Object {x: 1, y: Object}
````
## 6.10对象方法
Object.prototype里常见的方法，例如toString(),valueOf()等，但有些特定的类会重写这些方法。
