---
title: FormData笔记
date: 2017-05-06
tags: ['FormData']
categories: ['笔记']
---
之前项目里需要用到传很多参数的表单提交，原生的表单提交过程可控性差，又不想手动拼接很长的参数，于是决定学习和使用FormData进行替代。
<!--more-->
# 概念
XMLHttpRequest Level 2添加了一个新的接口FormData.利用FormData对象,我们可以通过JavaScript用一些键值对来模拟一系列表单控件,我们还可以使用XMLHttpRequest的send()方法来异步的提交这个"表单".比起普通的ajax,使用FormData的最大优点就是我们可以异步上传一个二进制文件。
#基本使用
1. 使用new FormData()插件一个FormData对象，调用append()添加字段。
````javascript
var formData = new FormData();

formData.append("username", "Groucho");
formData.append("accountnum", 123456); // 数字 123456 会被立即转换成字符串 "123456"

// HTML 文件类型input，由用户选择
formData.append("userfile", fileInputElement.files[0]);

// JavaScript file-like 对象
var content = '<a id="a"><b id="b">hey!</b></a>'; // 新文件的正文...
var blob = new Blob([content], { type: "text/xml"});

formData.append("webmasterfile", blob);

var request = new XMLHttpRequest();
request.open("POST", "http://foo.com/submitform.php");
request.send(formData);
````
2. 构造一个包含Form表单数据的FormData对象，需要在创建FormData对象时传入指定表单的元素
```javascript
var formElement = document.querySelector("form");
var formData = new FormData(formElement);
var request = new XMLHttpRequest();
request.open("POST", "submitform.php");
formData.append("serialnumber", serialNumber++);
request.send(formData);
````
3. 常用api
* append() 用于添加一个新值到 FormData 对象内的一个已存在的键中，如果键不存在则会添加该键。
* delete() 用于从 FormData 对象中删除指定 key 和它对应的 value(s)。
* entries() 用于返回一个 iterator对象 ，此对象可以遍历访问FormData中的键值对。其中键值对的key是一个 USVString 对象；value是一个 USVString , 或者 Blob对象。
# 配合axios使用
之前项目是使用axios进行http请求，FormData配合axios使用，请求方法和请求头配置为
````javascript
axios.post(url, params, {
    method: 'post',
    headers: { 'Content-Type': 'multipart/form-data' }
})
````
# 参考资料
[https://developer.mozilla.org/zh-CN/docs/Web/API/FormData](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)
