---
title: JavaScript的继承
date: 2017-12-16
tags: ['inherit', 'js']
categories: ['笔记']
---
## 原型链继承
原理： 将父类的实例作为子类的原型。
<!--more-->

````javascript
function Parent() {
  this.name = 'Tom'
}

Parent.prototype.getName = function() {
  return this.name
}

function Child() {

}

Child.prototype = new Parent()

//demo
var child1 = new Child()
console.log(child1.getName()) // 'Tom'
````

缺点：

1. 原型中引用类型的属性被所有实例共享，改变一个会影响另一个实例的属性，存在篡改的可能。
2. 在创建 Child 的实例时，不能向Parent传参。

````javascript
function Parent () {
    this.names = ['Tom', 'Jack'];
}

function Child () {

}

Child.prototype = new Parent();

//demo
var child1 = new Child();

child1.names.push('Danny');

console.log(child1.names); // ["Tom", "Jack", "Danny"]

var child2 = new Child();

console.log(child2.names); // ["Tom", "Jack", "Danny"]
````

## 借用构造函数继承

原理：使用父类的构造函数来增强子类实例，等同于复制父类的实例给子类（不使用原型）。

借用构造函数继承的核心就在于Parent.call(this, name)，“借调”了Parent构造函数，这样，Child的每个实例都会将Parent中的属性复制一份。

````javascript
function Parent(name) {
  this.name = name
  this.colors = ["red", "blue"]
}

function Child(name, age) {
  Parent.call(this, name)
  this.age = age
}

//demo
var child1 = new Child('Jack', 10)
child1.colors.push("green")
console.log(child1.name) // Jack
console.log(child1.age) // 10
console.log(child1.colors) // ["red", "blue", "green"]

var child2 = new Child('Tom', 20)
console.log(child2.name) // Tom
console.log(child2.age) // 20
console.log(child2.colors) // ["red", "blue"]
````

优点：

1. 避免了引用类型的属性被所有实例共享。
2. 可以在 Child 中向 Parent 传参。

缺点：

1. 只能继承父类的实例属性和方法，不能继承原型属性/方法。
2. 无法实现复用，方法都在构造函数中定义，每次创建实例都会创建一遍方法，每个子类都有父类实例函数的副本，影响性能。

## 组合继承

原理：结合原型链继承和构造函数继承通过调用父类构造，继承父类的属性并保留传参的优点，然后通过将父类实例作为子类原型，实现函数复用。

其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承，这样，既通过在原型上定义方法实现了函数复用，又能保证每个实例都有它自己的属性。

````javascript
function Parent(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
  
}

Parent.prototype.getName = function() {
  return this.name
}

Parent.prototype.getAge = function() {
  return this.age
}

function Child(name, age) {
  Parent.call(this, name)
  this.age = age
}

Child.prototype = new Parent()
Child.prototype.constructor = Child

//demo
var child1 = new Child('Tom', 20)
child1.colors.push('yellow')
console.log(child1.colors) // ["red", "blue", "green", "yellow"]
console.log(child1.getName()) // Tom
console.log(child1.getAge()) //20

var child2 = new Child('Jack', 10)
child2.colors.push('pink')
console.log(child2.colors) // ["red", "blue", "green", "pink"]
console.log(child2.getName()) //Jack
console.log(child2.getAge()) //10
````

优点：
1. 融合原型链继承和构造函数的优点。

缺点：

1. 父类中的实例属性和方法既存在于子类的实例中（复制父类实例而来），又存在于子类的原型中（继承父类实例而来）。因此，在使用子类创建实例对象时，会调用两次父构造函数，其原型中会存在两份相同的属性/方法。
## 原型式继承

原理：直接将某个对象直接赋值给构造函数的原型。

object()对传入其中的对象执行了一次浅复制，将F的原型直接指向传入的对象

````javascript
function object(obj) {
  function F() {}
  F.prototype = obj
  return new F()
}

//demo
var person = {
  name: 'Tom',
  colors: ['red', 'green']
}

var person1 = object(person)
person1.name = 'Jack'
person1.colors.push('blue')

var person2 = object(person)
person2.name = 'Danny'
person1.colors.push('yellow')

console.log(person1.colors) // ["red", "green", "blue", "yellow"]
console.log(person2.colors) // ["red", "green", "blue", "yellow"]
console.log(person.colors) // ["red", "green", "blue", "yellow"]

````

缺点：

1. 原型链继承多个实例的引用类型属性指向相同，存在篡改的可能。
2. 无法传递参数。

另外，ES5中存在Object.create()的方法，能够代替上面的object方法。
## 寄生式继承

原理：在原型式继承的基础上，增强对象，返回构造函数。

````javascript
function createObj(o) {
  var clone = object(o) //通过调用函数创建一个新对象
  clone.sayHi = function() { //以某种方式增强这个对象
    return 'Hi'
  }
  return clone
}

//demo
var person = {
  name: 'Tom',
  colors: ['red', 'green']
}

var person1 = createObj(person)
console.log(person1.name) // Tom
console.log(person1.colors) // ["red", "green"]
console.log(person1.sayHi()) // Hi
````

缺点：

1. 原型链继承多个实例的引用类型属性指向相同，存在篡改的可能
2. 无法传递参数

## 组合寄生式继承
原理：结合借用构造函数传递参数和寄生模式实现继承。

````javascript
function object(o) {
  function F() {}
  F.prototype = o
  return new F()
}

function inherit(child, parent) {
  var prototype = object(parent.prototype) //创建对象
  prototype.constructor = child // 增强对象
  child.prototype = prototype // 指定对象
}


//demo
function Parent(name) {
  this.name = name
  this.colors = ['red', 'green']
}
Parent.prototype.getName = function() {
  return this.name
}

function Child(name, age) {
  Parent.call(this, name)
  this.age = age
}

inherit(Child, Parent)

Child.prototype.getAge = function() {
  return this.age
}

var child1 = new Child('Tom', 20)
child1.colors.push('yellow')
console.log(child1.colors) // ["red", "blue", "green", "yellow"]
console.log(child1.getName()) // Tom
console.log(child1.getAge()) //20

var child2 = new Child('Jack', 10)
child2.colors.push('pink')
console.log(child2.colors) // ["red", "blue", "green", "pink"]
console.log(child2.getName()) //Jack
console.log(child2.getAge()) //10
````

优点：

1. 只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。
2. 原型链能保持不变，能够正常使用 instanceof 和 isPrototypeOf。
### 参考资料
* [《javascript高级程序设计》笔记：继承](https://segmentfault.com/a/1190000011917606)
* [JavaScript深入之继承的多种方式和优缺点](https://github.com/mqyqingfeng/Blog/issues/16)