---
title: html标签语意化的区别
date: 2017-10-20
tags: ['html','语意化']
categories: ['笔记']
---
## title 和 H1 
* title

HTML \<title\> 元素 定义文档的标题，显示在浏览器的标题栏或标签页上。它只可以包含文本，若是包含有标签，则包含的任何标签都不会被解释。
<!--more-->
* H1
标题(Heading)元素拥有六个不同的级别，\<h1\> 是最高级的，而 \<h6\> 则是最低的级别。 一个标题元素能简要描述该节的主题。
## b 和 strong
* b

HTML \<b\>元素表表示相对于普通文本字体上的区别，但不表示任何特殊的强调或者关联，通常以粗体显示。

* strong

Strong 元素 (\<strong\>)表示文本十分重要，一般用粗体显示。

## i 和 em
* i

HTML元素 \<i\> 用于表现因某些原因需要区分普通文本的一系列文本。例如技术术语、外文短语或是小说中人物的思想活动等，它的内容通常以斜体显示。
* em

HTML 着重元素 (\<em\>) 标记出需要用户着重阅读的内容， \<em\> 元素是可以嵌套的，嵌套层次越深，则其包含的内容被认定为越需要着重阅读。
## 补充
> 在HTML4.01中： 
> < b >  < i > 是视觉要素（presentationl elements），分别表示无意义的加粗，无意义的斜体，表现样式为 { font-weight: bolder }，仅仅表示「这里应该用粗体显示」或者「这里应该用斜体显示」，此两个标签在HTML4.01中并不被推荐使用；
> 
> < em >  和  < strong >  是表达要素(phrase elements)。 < em > （emphasized text）表示一般的强调文本，而 < strong > （strong emphasized text）表示比 < em > 语义更强的的强调文本。
> 
> 而在新的 HTML5 工作草案 中：
> 
> < em >  和  < strong >  仍旧是表达要素(phrase elements)。但这时的 < strong > 表示html页面上的强调（emphasized text）， < em > 表示句子中的强调（即强调语义）
>
> 从规范中可以注意到：b 和 i 元素将被赋予真正的语义。更应有预见性注意 b 、i 与 strong 、em 的不同使用 。

## 参考资料
* [https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element)
* [https://www.zhihu.com/question/19551271/answer/12521687](https://www.zhihu.com/question/19551271/answer/12521687)
