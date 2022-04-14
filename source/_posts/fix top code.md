---
title: 吸顶效果常用实现代码
date: 2020-01-21
tags: ["吸顶"]
categories: [""]
---

## 吸顶效果常用实现代码

## 原理：通过监听吸顶元素的滚动位置来切换样式。

## 代码

```javascript
const el = this.$refs.card;
const offsetTop = el && el.getBoundingClientRect().top;
const elHeight = el && el.clientHeight;
if (-offsetTop > elHeight) {
  this.isTabsStick = true;
} else {
  this.isTabsStick = false;
}
```
