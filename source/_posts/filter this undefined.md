---
title: `this` undefined in vue filters 原因及解决办法
date: 2017-04-18
tags: ["vue"]
categories: ["vue"]
---

## 表现

在 vue filter function 获取 this，得到 undefined，无法获取到 vue 实例上下文。

## 原因

vuw 2.x 将 filter 设计为纯函数，不依赖 this 上下文。 [来源](https://github.com/vuejs/vue/issues/5998)

## 解决办法

1. 使用 computed 或者 methods 替代，filter 应该是纯函数，不应该依赖外界或者对外界有所影响。（vue 作者推荐）

2. 将需要用到的 this 上下文数据作为参数，传进 filter function。

3. 在 created 中局部声明 that 指向 this 实例对象，在 filters 中用 that 作为替代 this 使用。

4. 修改 Vue 原型上的 \_f 函数，修改 this 的指向。

```javascript
  created () {
    this._f = function (id) {
      return this.$options.filters[id].bind(this._self)
    }
  }
```

其中 this.$options 是 Vue 将所有选项包括 filters 选项进行合并，并生成一个最终的选项对象$options。而获取 filter 方法的 \_f 函数，原本的实现 this 并没有绑定 vue 实例的上下文，所以在 filter 中会拿不到 vue 实例上下文。
