---
title: ES6 module 中使用 export default 的缺点
date: 2019-05-18
tags: ["es6"]
categories: ["javascript"]
---

## 使用 export default 的缺点

## https://blog.neufund.org/why-we-have-banned-default-exports-and-you-should-do-the-same-d51fdc2cf2ad

### 更好的编辑器提示

使用 export default 不导出任何具体的名称。相反的，如果给导出的值指定具体的名称，可以使 IDE 可以自动查找和导入依赖项，这极大地提高了生产率。

### 更利于重构

export default 使大规模重构变得非常困难，因为每个引用的地方都可以对默认导入进行不同的命名，不利于维护。
<!--more-->
### 更利于摇树

导出一个具有很多属性的对象，这是一种反模式，显然不利于摇树。

```javascript
// do not try this
export default {
  propertyA: "A",
  propertyB: "B",
};
// do this instead
export const propertyA = "A";
export const propertyB = "B";
```

当引用者不使用所有导出的值时，使用命名的导出可以减小包的大小（在构建库时特别有用）。
