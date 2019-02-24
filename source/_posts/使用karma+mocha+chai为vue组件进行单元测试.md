---
title: 使用karma+mocha+chai为vue组件进行单元测试
date: 2017-06-24
tags: ['vue','karma','mocha','chai']
categories: ['测试']
---
最近学习前端的单元测试，并对vue组件进行了单元测试。
## 主要工具
### karma
Karma是一个基于Node.js的JavaScript测试执行过程管理工具（Test Runner）。

负责提供测试的浏览器环境，执行测试框架，已经生成覆盖率报告。

关于karma的配置
````javascript
var webpackConfig = require('../../build/webpack.test.conf')

module.exports = function (config) {
  config.set({
    // to run in additional browsers:
    // 1. install corresponding karma launcher
    //    http://karma-runner.github.io/0.13/config/browsers.html
    // 2. add it to the `browsers` array below.
    browsers: ['PhantomJS'],// 测试采用的浏览器环境
    frameworks: ['mocha', 'sinon-chai', 'phantomjs-shim'],// 使用的测试框架
    reporters: ['spec', 'coverage'],// 设定报告输出插件
    files: ['./index.js'],// 执行文件入口，（如果是使用vue-cli生成的index.js，会去读specs文件夹下的测试代码）
    preprocessors: {
      './index.js': ['webpack', 'sourcemap']
    },// 预处理器，测试之前对文件进行处理less编译，coffeeScript编译之类的，这里是加入webpack与sourcemap插件
    webpack: webpackConfig, // webpack配置
    webpackMiddleware: {
      noInfo: true
    }, // webpack中间件配置，可配置不输出信息，或者只输出错误信息等
    coverageReporter: {
      dir: './coverage',
      reporters: [
        { type: 'lcov', subdir: '.' },
        { type: 'text-summary' }
      ]
    }// 生成覆盖率报告的配置
  })
}
````
### mocha
mocha是一个Javascript测试框架，用来运行测试。
mocha使用多个describe块来展示不同的组件，然后在其中添加it块来制定一个单一的测试。
### chai
chai支持BDD风格和TDD风格的断言库，可以和任何的javascript测试框架配合使用。

'断言'就是判断源码的实际执行结果与预期结果是否一致，如果不一致就抛出一个错误。

下面的例子采用expect的断言风格。

TDD指的是Test Driven Development，测试驱动开发，先编写单元测试代码，再编写代码实现功能，运行单元测试代码，循环此过程，直到整个单元测试都通过。

BDD指的是Behavior Driven Development，行为驱动开发，是一种敏捷软件开发的技术，它鼓励软件项目中的开发者、QA和非技术人员或商业参与者之间的协作，是从用户的需求出发，强调系统行为。
## 对vue分页组件进行单元测试
项目采用vue-cli生成的，初始化的时候把测试相关的选项都选是。

vue-cli生成的目录结构里，和测试相关的是test文件夹。
````javascript
├─test
   ├─e2e // 端到端测试
   │
   │
   └─unit // 单元测试
      ├─specs
      │    └─Pagination.spec.js // 组件对应的测试代码
      │──.eslintrc // eslint配置
      │──idnex.js // karma执行的入口文件
      │──utils.js // 工具方法
      └──karma.conf.js // karma配置
````

### 创建vue的组件实例
根据[vue官方文档对单元测试的介绍](https://cn.vuejs.org/v2/guide/unit-testing.html)，可以使用Vue.extend()创建一个实例并检查渲染输出,同时使用$mount()进行挂载。

将该方法进行封装：
````javascript
import Vue from 'vue'

export const createCompInstance = (Component, propsData) => {
  const Constructor = Vue.extend(Component)
  return new Constructor({
    propsData
  }).$mount()
}

````
### 针对分页组件编写的单元测试
测试组件被创建成功。

通过exist断言判断组件实例是否创建，挂载。
````javascript
let vm
it('create', () => {
  vm = createCompInstance(Pagination, {
    totalNum: 100
  })
  const elm = vm.$el
  // prev
  expect(elm.querySelector('.prev')).to.exist
  // pager
  expect(elm.querySelector('.pager')).to.exist
  // next
  expect(elm.querySelector('.next')).to.exist
  // jumper
  expect(elm.querySelector('input')).to.exist
  expect(elm.querySelector('.btn')).to.exist
})
````
测试页码的点击。

获取组件挂载的DOM进行点击，再进行equal断言，判断当页码是否已经改变。
````javascript
it('event: click page', () => {
  vm = createCompInstance(Pagination, {
    pageSizes: [5, 10, 20],
    buttonCount: 10,
    totalNum: 100
  })
  const elm = vm.$el

  // click 3
  elm.querySelectorAll('.pager')[2].click()

  setTimeout(() => {
    expect(elm.querySelectorAll('.pager')[2].className).to.equal('current')
    expect(vm.currentPage).to.equal(3)
    done()
  }, 100)
})
````
实现按钮点击：通过获取相关元素el，执行 el.click() 方法实现。

> 由于 Vue 进行 异步更新DOM 的情况，一些依赖DOM更新结果的断言必须在 Vue.nextTick 回调中进行。

涉及到 Vue.nextTick 和 setTimeout 等异步操作的测试，要使用 done() 来标记完成时间点。

在describe的回调函数传入done，之后才能进行调用。

一开始的测试覆盖率很低，经过不断的调整和补充测试用例，最后的测试结果如下：
![test result](http://oq8q06ybp.bkt.clouddn.com/image/unittest.PNG)
以及测试覆盖率报告
![test coverage](http://oq8q06ybp.bkt.clouddn.com/image/testCoverage.PNG)
结果测试的覆盖率还是没有达到百分之百，好像还有一些分支条件没有覆盖到，得继续完善。
## 参考资料
1. [karma](http://karma-runner.github.io)
2. [mocha](https://mochajs.org/)
3. [chai](http://chaijs.com/)
4. [vue单元测试](https://cn.vuejs.org/v2/guide/unit-testing.html)