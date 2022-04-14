---
title: 使用 css 伪类实现多语言样式
date: 2020-11-18
tags: ["css"， "i18n"]
categories: [""]
---

## 为 html 标签加上对应语言码的 lang 属性

```javascript
document.getElementsByTagName("html")[0].setAttribute("lang", useLang);
```

## 在 css 中通过伪类来控制不同语言的样式

```css
.text {
  color: #5cb8ff;

  :lang(ar) & {
    // 阿拉伯语（这里要用标准的语言码）
    color: #c0662c;
  }
}
```

## 阿拉伯（文字从右到左）等语言环境下的镜像处理

1. 在 html 元素上增加属性 dir=rtl
2. 尽可能使用 flex 布局
3. margin-left, padding-left 等区分左右的属性，需要手动进行切换样式

```css
.text {
  padding-left: 10px;

  :lang(ar) & {
    // 阿拉伯语（这里要用标准的语言码）
    padding-left: 0px;
    padding-right: 10px;
  }
}
```