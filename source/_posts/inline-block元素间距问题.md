---
title: inline-block元素间距问题
date: 2017-11-05
tags: ['css']
categories: ['css']
---
## 背景
有时候，我们需要将&lt;li&gt;横向排列，而又为了能设置其宽度和高度，为其设置display:inline-block，相邻&lt;li&gt;之间会出现8px的空白间隔，不是margin也不是padding。
<!--more-->
## 产生原因
浏览器会把inline元素间的空白字符（空格、换行、Tab等）渲染成一个空格。而为了美观。我们通常是一个&lt;li&gt;放在一行，这导致&lt;li&gt;换行后产生换行字符，它变成一个空格，占用了一个字符的宽度。
> 此乃历史问题。不难理解，空白字符压缩（white space collapse）是西文排版的必然结果。

## 解决方式
1. 改变代码书写方式。

元素间留白间距出现的原因就是标签段之间的空格，因此，去掉HTML中的空格，自然间距就消失了。
````html
<ul><li>1</li><li>2</li><li>3</li></ul>
````
或者
````html
<ul>
  <li>1</li
  ><li>2</li
  ><li>3</li>
</ul>
````
2. 修改font-size

将&lt;ul&gt;内的字符尺寸直接设为0，即font-size: 0。不足：&lt;ul&gt;中的其他字符尺寸也被设为0，需要额外重新设定其他字符尺寸，且在Safari浏览器依然会出现空白间隔。

3. 使用margin负值

margin负值的大小与上下文的字体和文字大小相关，Arial字体的margin负值为-3像素，Tahoma和Verdana就是-4像素，而Geneva为-6像素。由于外部环境的不确定性，以及最后一个元素多出的父margin值等问题，这个方法不适合大规模使用。

4. 使用word-spacing或letter-spacing

letter-spacing子元素要设置letter-spacing为0，不然会继承父元素的值；使用word-spacing时，只需设置父元素word-spacing为合适值即可。

## 参考资料
* [有哪些好方法能处理 display: inline-block 元素之间出现的空格？
](https://www.zhihu.com/question/21468450)
* [inline-block间距产生的原因，去除inline-block间距的方法？](https://www.jianshu.com/p/b6fb427308ad)
