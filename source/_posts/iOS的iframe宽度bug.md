---
title: iOS的iframe宽度bug
date: 2017-09-30
tags: ['iOS','iframe','bug']
categories: ['工作总结']
---
## 问题背景
最近做一个项目里，有一个使用了[slick](https://github.com/kenwheeler/slick)插件做轮播效果的页面，本身打开一切正常。后面这个页面被嵌在另一个页面的iframe下时，iframe的宽度设置为100%，在iOS里打开，页面就变得不正常了，轮播部分好像被放得很大，把其他元素挤出屏幕外，轮播来回切换，页面来回闪烁。
<!--more-->
## 产生原因
iOS下的safari是按照iframe里面页面元素的全尺寸来调整iframe的大小的。

轮播效果的实现原理是将所有的幻灯片都放到一个块里面，然后设置溢出隐藏，在iOS的iframe下，结合上面第一条原因，就会将iframe撑得很大。
## 解决方案
### 方案一
在包含轮播组件的元素上设置以下样式：
````css
{
  width: 1px;
  min-width: 100%;
}
````
或者设置
````css
{
  width: 100vw;
}
````
### 方案二
在iframe上设置以下样式
````css
iframe {
  width: 1px;
  min-width: 100%;
  *width: 100%;
}
````
同时设置scrolling属性为'no'。
````html
<iframe height="950" width="100%" scrolling="no" src="Content.html"></iframe>
````
## 参考资料
* [https://www.spacevatican.org/2015/4/7/on-mobile-safari-and-iframes/](https://www.spacevatican.org/2015/4/7/on-mobile-safari-and-iframes/)
* [https://stackoverflow.com/questions/20123960/how-to-get-iframe-width-100-in-iphone-portrait-view](https://stackoverflow.com/questions/20123960/how-to-get-iframe-width-100-in-iphone-portrait-view)
* [https://stackoverflow.com/questions/23083462/how-to-get-an-iframe-to-be-responsive-in-ios-safari](https://stackoverflow.com/questions/23083462/how-to-get-an-iframe-to-be-responsive-in-ios-safari)
* [https://github.com/kenwheeler/slick/issues/1470](https://github.com/kenwheeler/slick/issues/1470)
