---
title: vue项目总结
date: 2017-06-10
tags: ['vue']
categories: ['工作总结']
---
1. 项目结构: 使用vue官方手脚架生成初始结构，src目录结构修改为
````javascript
│  App.vue
│  main.js
│
├─assets
│  ├─css
│  │      style.css 
│  │
│  └─images
│          product.jpg
│
├─components
│      Head.vue
│      Loading.vue
│
├─lib
│      api.js
│      mobileUtil.js
│
├─router
│      router.js
│
├─views
│  ├─pageA
│  │      childA.vue
│  │      childB.vue
│  │      Main.vue
│  │
│  ├─pageB
│  │      childA.vue
│  │      childB.vue
│  │      Main.vue
│  │
│  └─pageC
│         childA.vue
│         childB.vue
│         Main.vue
│  
│
└─vuex
    │  store.js
    │
    └─modules
        ├─pageA
        │      actions.js
        │      getters.js
        │      index.js
        │      mutation-type.js
        │      mutations.js
        │
        ├─pageB
        │      actions.js
        │      getters.js
        │      index.js
        │      mutation-type.js
        │      mutations.js
        │
        └─pageC
               actions.js
               getters.js
               index.js
               mutation-type.js
               mutations.js
````
assets文件夹存放img，css等静态资源。

components文件夹存放公共组件。

lib文件夹存放工具类，api配置等。

router文件夹存放路由配置。

views文件夹存放页面级的组件，统一Main.vue为每个页面主入口，同一页面的其他组件为子路由组件。

vuex文件夹存放vuex全局数据，store.js为主入口，里面的子文件夹按页面划分模块。

2. vuex的action间需要进行异步回调时，使用Promise。
3. <img>标签里可以通过require引入静态资源。
4. <keep-alive>配合activated钩子。
5. [动态路由](http://60kmlh.github.io/2017/07/08/基于vue-router的管理系统权限控制的实现/)
6. 利用process.env.NODE_ENV管理生产环境的api和开发环境的api前缀。
7. 利用v-if强制更新一个组件，$forceUpdate不生效。
8. 表单校验使用v-validate插件。
