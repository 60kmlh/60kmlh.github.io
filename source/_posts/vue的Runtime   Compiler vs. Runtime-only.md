---
title: vue的Runtime Compiler vs Runtime-only
date: 2017-10-12
tags: ['vue']
categories: ['笔记']
---
在vue的官方文档中可以得知，vue有两种不同的构建方式，运行时+编译器（Runtime+Compiler）和只包含运行时（Runtime-only）。

两者的区别是：

运行时+编译器是完整版的构建，即vue.common.js,支持创建vue实例时，传入template选项。编译器会将模板编译成渲染函数。

运行时构建的vue版本删除了模板编译的功能，因此无法支持带template属性的Vue实例选项。使用该构建方式的vue需要借助vue-loader或者vueify事先将.vue文件内部的模板会在构建时预编译成渲染函数，供vue实例使用。

> 因为运行时构建相比完整版缩减了30%的体积，你应该尽可能使用这个版本。

Vue的npm包也将package.json中的main指向了运行时构建dist/vue.runtime.common.js。

当你使用了运行时构建的版本，而由没有将模板预先编译成渲染函数，vue就会报如下的警告：
> [Vue warn]: Failed to mount component: template or render function not defined. (found in root instance).
## 参考资料
* [https://cn.vuejs.org/v2/guide/installation.html#运行时-编译器-vs-只包含运行时](https://cn.vuejs.org/v2/guide/installation.html#运行时-编译器-vs-只包含运行时)
* [https://zhuanlan.zhihu.com/p/25486761](https://zhuanlan.zhihu.com/p/25486761)
