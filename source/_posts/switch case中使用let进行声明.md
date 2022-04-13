---
title: switch 语句内的块级作用域
date: 2017-04-18
tags: ["switch/case"]
categories: ["javascript"]
---

## 现象

在 switch/case 中使用 let 进行声明，ESLint 会报错：SyntaxError: Identifier 'msg' has already been declared

```javascript
switch (key) {
  case value1:
    let a = 1;
    break;
  case valu2:
    let a = 2;
    break;
  default:
    break;
}
```

这是因为第一个 let a = 1; 与第二个 la = 2; 语句产生了冲突，虽然他们处于各自分隔的 case 语句中，即 case value1: 和 case value2:。导致这一问题的根本原因在于两个 let 语句处于同一个块级作用域，所以它们被认为是同一个变量名的重复声明

## 解决办法

把 case 语句加上花框{ }, 明確的告訴 ESLint 不同 case 的作用域也不同，则可以解决这个问题。

```javascript
switch (key) {
  case value1: {
    let a = 1;
    break;
  }
  case valu2: {
    let a = 2;
    break;
  }
  default:
    break;
}
```

## 参考资料

[禁止在 case 或 default 子句中出现词法声明 (no-case-declarations)
](https://cn.eslint.org/docs/rules/no-case-declarations)
