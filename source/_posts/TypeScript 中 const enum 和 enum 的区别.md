---
title: TypeScript 中 const enum 和 enum 的区别
date: 2021-11-13
tags: ["typescript"]
categories: ["typescript"]
---

## const enum 编译时会把实际用到的枚举成员替换成常量值

```ts
// const-enum.ts
const enum Color {
  Red,
  Green,
  Blue,
}
console.log(Color.Red);

// output.js
console.log(0 /* Red */);
```
<!--more-->
## enum 会编译成运行时的对象

```ts
// enum.ts
enum Color {
  Red,
  Green,
  Blue,
}
console.log(Color.Red);

// output.js
var Color;
(function (Color) {
  Color[(Color["Red"] = 0)] = "Red";
  Color[(Color["Green"] = 1)] = "Green";
  Color[(Color["Blue"] = 2)] = "Blue";
})(Color || (Color = {}));
console.log(Color.Red);
```

## 不同场景下的选择

- 尽可能使用 const enum，编译后代码体积较小。

- 在需要动态获取值的场景下，只能使用 enum ，const enum 不能获取到值。

```ts
const enum Color {
  Red,
  Green,
  Blue,
}

// 这样可以
console.log(Color.Blue);

// 这样不行!!
const key = "Blue";
console.log(Color[key]);
```

- 编写第三方库的时候，采用 const enum，兼容使用者不同的使用场景。
