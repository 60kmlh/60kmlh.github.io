---
title: Google Drive 文件通过 js 下载的方法
date: 2020-05-09
tags: ["google drive"]
categories: [""]
---

## Google Docs: 分享的文档链接形式为：

https://docs.google.com/document/d/FILE_ID/edit?usp=sharing

## 将 /edit 用 /export 替换掉，然后添加上文件的格式，例如 doc，xlsx，pdf 等，然后该文件就能被保存了。

https://docs.google.com/document/d/FILE_ID/export?format=xlsx
<!--more-->
## Google 电子表格: 公开的电子表格链接格式为：

https://docs.google.com/spreadsheets/d/FILE_ID/edit?usp=sharing

其下载链接格式为：https://docs.google.com/spreadsheets/d/FILE_ID/export?format=xlsx

## Google 演示文稿: 演示文稿的分享链接形式是这样的格式：

https://docs.google.com/presentation/d/FILE_ID/edit?usp=sharing

如果下载 PPT（.pptx）和 PDF 格式的链接如下：

https://docs.google.com/presentation/d/FILE_ID/export/pptx

https://docs.google.com/presentation/d/FILE_ID/export/pdf

## 代码

```js
window.open("https://docs.google.com/spreadsheets/d/FILE_ID/export?format=xlsx ", "_blank");
```
