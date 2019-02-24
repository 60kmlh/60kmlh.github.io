---
title: 使用react和vue实现支付宝微信风格的密码输入框
date: 2017-10-08
tags: ['react','vue']
categories: ['公共组件']
---
# 需求背景
最近的一个项目需要实现一个类似支付宝密码输入的验证码框，即确定好格子的数量，让用户输入密码或者验证码。

在此基础上进一步扩展，实现具有以下功能的公共组件：
1. 仿支付宝微信密码输入的输入方式，即确定好格子的数量，用户输入。
2. 支持自定义格子数，4位到8位。
3. 输入格子可显示出明文或者使用密码黑点代替。
4. 支持调起数字键盘或混合键盘。
5. 自定义标题。
6. 支持按钮提交或者输入完成时自动提交
# 实现过程
## 基本思路
这次实现的是mvvm框架(react和vue)下的组件，通过设置一个隐藏的输入框，借助mvvm框架的数据绑定功能，将用户输入的值分别映射到每个格子上。

格子本身不是给用户直接输入的，用户可直接输入的只有隐藏的输入框。
## 样式
1. 隐藏输入框
首先设置position：absolute，让input元素脱离文档流，不占据空间。

接着将input元素设置为透明，这个时候在电脑上查看是隐藏的，但是手机查看，输入的内容是隐藏了，但还是会有一个闪烁的光标。

后面将input元素的margin-left的值设置为一个很大的值，直接将这个input元素挤出屏幕外，这样就彻底看不到了。
2. 格子
格子的排列采用flex局部，将父元素设置为display: flex，同时将justify-content设置为space-around，这样将格子均匀地分布。

格子的数量在4个到8个之间，格子的长宽采用em作单位，同时值的大小根据格子的数量动态进行调整，实现当格子的数量增多时，格子的大小会相对进行缩小。

格子得满足实现密码输入时显示黑点，因此格子采用input元素来实现，设置disable为true，不可输入。这里有个注意的点，disabled的input元素opacity默认是0.3,所以样式里要加上opacity:1。
3. 输入时页面放大
在手机下进行输入时，当隐藏的input元素被聚焦输入时，此时页面会被手机浏览器放大。

为了禁止这种行为，需要为页面增加以下meta
````html
<meta name="viewport" content="width=device-width,minimum-scale=1.0,maximum-scale=1.0,initial-scale=1.0,user-scalable=no">
````
width - viewport的宽度

initial-scale - 初始的缩放比例

minimum-scale - 允许用户缩放到的最小比例

maximum-scale - 允许用户缩放到的最大比例

user-scalable - 用户是否可以手动缩放
## 功能逻辑
1. 控制隐藏框的输入
根据规定输入的位数，监听隐藏框的输入事件，使用String的slice方法截取符合长度的值。
2. 自定义格子数 
通过设置组件的属性，组件内部获取到格子的数量进行渲染。
3. 格子显示明文或黑点
显示明文，是借助mvvm框架的数据绑定功能，将隐藏input的值按位置绑定到对应的格子上。

根据传入组件的属性，当需要显示黑点时，设置input的type属性为password。

在react中，根据传入的属性，是这样设置的
````html
<input type={this.props.isPassword ? 'password' : 'text'} />
````
但是在vue中使用类似的写法，会报如下错误
> v-model does not support dynamic input types
> 
> What this error is saying is that, if you dynamically change the input type being sent to the component, Vue will not update the input element to change its type.

原因是vue不支持动态设置input的type属性，解决方法是设置两个不同type的input，借助vue指令v-if和v-else切换。

4. 调起数字键盘或混合键盘
根据传入组件的属性，需要显示数字键盘时，设置隐藏input元素的type属性为tel。
````html
<input type={this.props.isNumber ? 'tel' : 'text'} />
````
vue里面遇到的问题同上。
# 组件属性和事件
## Props
Name | type | default | description
---:| --- | ---| ---
tip | String | '请输入密码' | 输入提示语
passwordLength | Number | 6 | 格子位数
isPassword | Boolean | true | 显示明文或者密码黑点
isNumber | Boolean | true | 调出数字键盘或者混合键盘
isBtnCtr | Boolean | true | 使用确认按钮或者自动提交

## Events
Name | isRequire | description
---:| --- | --- 
onConfirm | require | 输入完成时的回调函数
onClose | require | 关闭输入框时的回调函数
# 演示页面
[![手机扫描二维码查看](http://oq8q06ybp.bkt.clouddn.com/image/qrCode.png)](http://60kmlh.ink/vue-GridPassword/)
# 源码
[react版本](http://github.com/60kmlh/react-GridPassword/)

[vue版本](https://github.com/60kmlh/vue-GridPassword/)
