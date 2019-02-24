---
title: babel常用配置记录
date: 2017-08-27
tags: ['babel']
categories: ['笔记']
---
## 简介
babel是一个javascript语法的编译器，借助babel，我们可以使用新的javascript语法和api，编译成低版本的javascript语法，能被低版本的浏览器识别，解决浏览器的兼容性问题。
## 使用
使用babel主要是学会babel的配置,并安装相应插件,告诉babel我们需要编译成哪个版本的javascript。

babel的配置就是对presets和plugins进行配置，这样babel才能按照我们预想的进行工作。
### presets
预设，是预先设定好一系列转换标准的配置，可以当做是一系列插件（plugins）的集合。
#### 官方preset
1. env 

babel-preset-env是根据你所支持的环境，依据[compat-table](https://github.com/kangax/compat-table)自动确定您需要的Babel插件。

默认设置是会编译所有的ES2015+的新特性，行为和babel-preset-latest一样。

常用设置项：
* targets: { [string]: number }，默认{} 

设置需要支持的环境版本，例如chrome, opera, edge, firefox, safari, ie, ios, android, node, electron。

推荐写上特定的版本号，如node: "6.10"。

* targets.node: number | string | "current" | true

设置node环境的版本

* targets.browsers: Array<string> | string

设置支持符合相应条件的浏览器集合，使用[browserslist](https://github.com/ai/browserslist),如
````javascript
{
  "browserslist": [
    "> 1%",
    "last 2 versions"
  ]
}
````
其余配置项参考[babel-preset-env](https://babeljs.io/docs/plugins/preset-env/)

2. es2015

将es2015编译成es5。这个preset主要是包含了一系列转换es2015方法的插件
> check-es2015-constants
>
> transform-es2015-arrow-functions
>
> transform-es2015-block-scoped-functions
>
> transform-es2015-block-scoping
>
> transform-es2015-classes
>
> transform-es2015-computed-properties
>
> transform-es2015-destructuring
>
> transform-es2015-duplicate-keys
>
> transform-es2015-for-of
>
> transform-es2015-function-name
>
> transform-es2015-literals
>
> transform-es2015-modules-commonjs
>
> transform-es2015-object-super
>
> transform-es2015-parameters
>
> transform-es2015-shorthand-properties
>
> transform-es2015-spread
>
> transform-es2015-sticky-regex
>
> transform-es2015-template-literals
>
> transform-es2015-typeof-symbol
>
> transform-es2015-unicode-regex
>
> transform-regenerator

常用设置项：
* modules: "amd" | "umd" | "systemjs" | "commonjs" | false, defaults to "commonjs".
3. es2016

将es2016编译成es5
4. es2017

将es2017编译成es5
5. Latest preset

包含最新的preset，包括了es2015，es2016，es2017的预设。

已经被移除，使用preset-env代替。
6. react
专门对react推出的预设，包含：

preset-flow

syntax-jsx

transform-react-jsx

transform-react-display-name
7. Flow preset
包含transform-flow-strip-types插件，用来转换javascript静态类型检查(flow语法)。
####　Stage-X (试验性预设)
Stage-X预设是用来转换那些未被批准，未发布Javascript的语言特性。

Stage-X是按照javascript的语法提案阶段区分的。

TC39（Technical Committee 39）是一个推动JavaScript发展的委员会。每一项新特性，要最终纳入ECMAScript规范中，TC39拟定了一个处理过程，称为TC39 process。

其中共包含5个阶段，Stage 0 ~ Stage 4。
* Stage 0: strawman

一种推进ECMAScript发展的自由形式，任何TC39成员，或者注册为TC39贡献者的会员，都可以提交。

* Stage 1: proposal

该阶段产生一个正式的提案。

* Stage 2: draft

草案是规范的第一个版本，与最终标准中包含的特性不会有太大差别。

* Stage 3: candidate

候选阶段，获得具体实现和用户的反馈。

* Stage 4: finished

已经准备就绪，该特性会出现在年度发布的规范之中。

可以看到Stage-0包含的范围是最广的，一般使用选择babel-preset-stage-2。
### plugins
插件，专门针对某一个语法或某个单一功能的转换。
[插件列表](https://babeljs.io/docs/plugins/#transform-plugins)
#### 常用插件
* babel-plugin-transform-es2015-classes

用于转换ES2015的类到ES5。

* babel-plugin-transform-es2015-spread

转换ES2015的扩展运算符。

* babel-plugin-transform-react-jsx

转换jsx语法

* babel-plugin-transform-runtime

transform-runtime提供包含编译模块的工具函数。

babel在转换 ES2015 语法为 ES 5 的语法时，babel需要一些辅助函数，例如 _extend。babel默认会将这些辅助函数直接插入到每一个 js 文件里，这样文件多的时候，重复的辅助函数多，项目重复代码就很多。

所以 babel 提供了 transform-runtime 来将这些辅助函数都引入到一个单独的模块 babel-runtime 中，避免重复的代码。

Babel 默认只转换新的 JavaScript 语法，而不转换新的 API。例如，Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法（比如 Object.assign）都不会转译。如果想使用这些新的对象和方法，必须使用 babel-polyfill，为当前环境提供一个垫片。

像 Promise 这样的全局对象会污染全局命名空间，这就要求库的使用者自己提供 polyfill。这些 polyfill 一般在库和工具的使用说明中会提到，比如很多库都会有要求提供 es5 的 polyfill。在使用 babel-runtime 后，库和工具只要在 package.json 中增加依赖 babel-runtime，交给 babel-runtime 去引入 polyfill 就行了，所以使用transform-runtime另一个作用是为你的代码提供一个沙盒环境。

常用设置项：

helpers：boolean, defaults to true.

是否引入Babel的helpers函数。

polyfill：boolean, defaults to true.

是否使用不会污染全局的polyfill。

非实例方法的polyfill，如Object.assign，但是实例方法不支持，如"foobar".includes("foo")，这时候需要单独引入babel-polyfill
## 执行顺序
* 插件在预设之前执行。
* 插件的执行顺序是从前往后。
* 预设的执行顺序是从后往前。
## 参考资料
[transform-runtime](https://babeljs.io/docs/plugins/transform-runtime)
[https://segmentfault.com/q/1010000005596587](https://segmentfault.com/q/1010000005596587)
[https://www.vanadis.cn/2017/04/28/difference-between-babel-polyfill-and-babel-runtime/](https://www.vanadis.cn/2017/04/28/difference-between-babel-polyfill-and-babel-runtime/)
