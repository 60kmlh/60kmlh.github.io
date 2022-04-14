---
title: TypeScript 中对象通过中括号读取属性值报错的问题
date: 2021-07-15
tags: ["typescript"]
categories: [""]
---

## 报错代码

```javascript
const obj = {
  key1: "xxx",
};
function getKeyValue(arg: string) {
  return obj[arg];
}
```

该代码在 typescript 中会报错：Element implicitly has an 'any' type because expression of type 'string' can't be used to index。

## 原因

key 只声明为 string 的话，typescript 为了防止运行时传入了未知的 key 进行取值，返回了 undefined 报错，于是对这种写法进行警告。

## 解决办法

1. 在 tsconfig 中关闭针对该错误的检查，suppressImplicitAnyIndexErrors 设置为 true。

2. 将 key 声明为 obj 的属性

```javascript
    const obj = {
  key1: "xxx",
};
function getKeyValue(arg: keyof typeof obj) {
  return obj[arg];
}

```

3. 更加通用的写法，使用泛型

```javascript
const obj = {
  key1: "xxx",
};
function getKeyValue<O>(object:O, arg: keyof O) {
  return object[arg];
}

```

## 参考资料

[https://dev.to/mapleleaf/indexing-objects-in-typescript-1cgi](https://dev.to/mapleleaf/indexing-objects-in-typescript-1cgi)
[https://stackoverflow.com/questions/57086672/element-implicitly-has-an-any-type-because-expression-of-type-string-cant-b](https://stackoverflow.com/questions/57086672/element-implicitly-has-an-any-type-because-expression-of-type-string-cant-b)
