---
title: react服务端渲染的实现
date: 2018-08-11
tags: ['react', 'js', 'ssr']
categories: ['react']
---
## 什么时候需要服务端渲染（Server Side Render）
1. 现代前端应用的页面内容由ajax请求数据之后，再在客户端进行页面的动态渲染，而一般搜索引擎或网页爬虫只对ajax渲染前的HTML文件进行爬取，这种形式不利于网站的SEO。
2. 单页应用通过请求js和ajax请求数据之后，再进行页面结构和数据的渲染，降低了首屏的展示速度。
<!--more-->
因此，通过服务端渲染的方式，将渲染结果以HTML结构的形式返回给客户端，可以优化以上两个主要的问题。

## react中和SSR相关的API
* renderToString()

将 React 元素渲染为初始 HTML。React 将返回一个 HTML 字符串。你可以使用此方法在服务端生成 HTML。
* renderToStaticMarkup()

此方法与 renderToString 相似，但此方法不会在 React 内部创建的额外 DOM 属性，例如 data-reactroot。如果你希望把 React 当作静态页面生成器来使用，此方法会非常有用，因为去除额外的属性可以节省一些字节。

如果你计划在前端使用 React 以使得标记可交互，请不要使用此方法。你可以在服务端上使用 renderToString 或在前端上使用 ReactDOM.hydrate() 来代替此方法。

**以上两个API可以被使用在服务端和浏览器环境。**

**下述附加方法依赖一个只能在服务端使用的 package（stream）。它们在浏览器中不起作用。**
* renderToNodeStream()

将一个 React 元素渲染成其初始 HTML。返回一个可输出 HTML 字符串的可读流。通过可读流输出的 HTML 完全等同于 ReactDOMServer.renderToString 返回的 HTML。
* renderToStaticNodeStream()

此方法与 renderToNodeStream 相似，但此方法不会在 React 内部创建的额外 DOM 属性，例如 data-reactroot。如果你希望把 React 当作静态页面生成器来使用，此方法会非常有用，因为去除额外的属性可以节省一些字节。
## 实现

下面借助 renderToString() 方法实现一个简单的服务端渲染例子。
### 引入服务端框架
这里我们采用 express 框架。
新建一个 React 组件.

**App.js**
````javascript
import React, { Component } from 'react'

class App extends Component {
  constructor(props) {
    super(props)
    this.state = {
      count: 0
    }
  }
  
  render() {
    return (
      <div>
        首页
        <div>
          <button onClick={() => {this.setState({count: this.state.count + 1})}}>+</button>
          <div>{this.state.count}</div>
        </div>
      </div>
    )
  }
}

export default App
````
**server.js**

````javascript
import express from 'express'
import fs from 'fs'
import path from 'path'
import { renderToString } from 'react-dom/server'
import React from 'react'
import App from './src/App'

var app = express()

app.get('/*', (req, res) => {
  const renderedString = renderToString(
        <App/>
    )

  fs.readFile(path.resolve('index.html'), 'utf8', (error, data) => {
    if(error) {
      console.log(error)
      res.send('<p>Error!</p>')
      return false
    }
    console
    res.send(`${data.replace('<div id="app"></div>', `<div id="app">${renderedString}</div>`)}`)
  })
})

app.listen(8000)
````
然后运行 node server.js。这个时候会发现，启动报错。原因在于我们在 node 里面使用了 ES6 Module 和 React JSX，node并不支持。

引入 babel-node 来解决这个问题。

安装完成 babel-node 以及 babel-core，babel-cli，babel-preset-react 等相关工具之后，执行
````javascript
"scripts": {
  "start": "babel-node ./server.js --presets es2015,stage-0,react"
},
````
访问 localhost:8000,就可以看到页面效果了。

![服务端渲染](https://i.loli.net/2019/03/22/5c944d96a9ea0.png)
### 客户端React组件的实例化
上面的App组件中，点击 + 按钮，并不会生效，这个页面只是一个静态的HTML页面，没有在客户端渲染React组件并初始化React实例。只有在初始化React实例后，才能更新组件的state和props，初始化React的事件系统，执行虚拟DOM的重新渲染机制。

那么这里可能会有疑问，服务器端已经渲染了一次React组件，如果在客户端中再渲染一次 React 组件，会不会渲染两次 React 组件。答案当然是不会的。

在 React16 版本以前，如果使用renderToString渲染组件，会在组件的第一个DOM带有data-react-checksum属性，当客户端渲染React组件时，首先计算出组件的checksum值，然后检索HTML DOM看看是否存在数值相同的data-react-checksum属性，如果存在，则组件只会渲染一次，如果不存在，则会抛出一个warning异常。

也就是说，当服务器端和客户端渲染具有相同的props和相同DOM结构的组件时，该React组件只会渲染一次。

在服务器端使用renderToStaticMarkup渲染的组件不会带有data-react-checksum属性，此时客户端会重新渲染组件，覆盖掉服务器端的组件。因此，当页面不是渲染一个静态的页面时，最好还是使用renderToString方法。

接着根据 data-reactid 属性，找到需要绑定的事件元素，进行事件绑定的处理。

> React v16 版本里，ReactDOMServer 渲染的内容不再有 data-react 的属性，而是尽可能复用 SSR 的 HTML 结构。这就带来了一个问题，ReactDOM.render 不再能够简单地用 data-react-checksum 的存在性来判断是否应该尝试复用，如果每次 ReactDOM.render 都要尽可能尝试复用，性能和语义都会出现问题。所以， ReactDOM 提供了一个新的 API， ReactDOM.hydrate() 。
> 在 React v17 版本里，ReactDOM.render 则直接不再具有复用 SSR 内容的功能。

实践发现，v16 版本里面，render() 方法会将整个 DOM 结构重新渲染， 而使用 hydrate 则不会。

加上在客户端渲染的代码
**index.js**

````javascript
import React from 'react'
import ReactDOM from 'react-dom'
import { BrowserRouter } from 'react-router-dom'
import App from './App'

ReactDOM.render(
  <App />
, document.getElementById('app'))
````
这个时候，我们需要引入 webpack，将要在客户端执行的代码用 webpack 打包到 dist 文件夹。

````javascript
"scripts": {
  "start": "npm run build && babel-node ./server.js --presets es2015,stage-0,react",
  "build": "webpack"
},
````
在 html 模板里手动引入js，
````html
<body>
  <div id="app"></div>
</body>
<script src="dist/app.js"></script>
````
再次启动项目，这个时候会发现，请求回来的 app.js 的内容变成了 html.index 文件。

![服务端渲染](https://i.loli.net/2019/03/22/5c944e3cb98ad.png)
原因在于请求 /dist/app.js 被当成了普通的路由了，没有被当成一个静态资源来返回有效的 JavaScript 代码，解决方案就是在 express 中添加一个静态资源服务。
````javascript
var app = express()
app.use(express.static('dist'));
````
同时将 html 的 script 路径修改下，
````html
<body>
  <div id="app"></div>
</body>
<script src="/app.js"></script>
````
这样就可以客户端就可以正确地获取到 app.js 的内容了，然后在客户端进行 React 组件的实例化，这个时候点击 + 按钮也有效果了。
### 路由
接下来添加路由功能

服务端匹配路由的时候，不能用 BrowserRouter，要使用无状态的 StaticRouter，并结合 location 和 context 两个属性。

给App组件添加路由。

**App.js**
````javascript
import { Switch, Route, Link } from 'react-router-dom' 
import PageA from './component/PageA'
import PageB from './component/PageB'
//...
  render() {
    return (
      <div>
        首页
        <div>
          <button onClick={() => {this.setState({count: this.state.count + 1})}}>+</button>
          <div>{this.state.count}</div>
        </div>
        <Link to='/'>跳转到pageA</Link>
        <br/>
        <Link to='/pageb'>跳转到pageB</Link>
        <Switch>
          <Route exact path="/" component={ PageA } />
          <Route exact path="/pageb" component={ PageB } />
        </Switch>
      </div>
    )
  }
//...
````
服务端加入路由匹配。这里默认浏览器 url 和 配置的路由地址一样，也可以把路由抽成单独的配置文件，借助 react-router-dom 路由模块的 matchPath 方法来匹配路由。

**server.js**
````javascript
import { StaticRouter } from 'react-router-dom'

app.get('/*', (req, res) => {
  const renderedString = renderToString(
    <StaticRouter location={ req.url }>
      <App />
    </StaticRouter>
  )
  //...
})
````
可以看到，已经实现了路由功能。

![服务端渲染](https://i.loli.net/2019/03/22/5c9455e9e0e31.png)
![服务端渲染](https://i.loli.net/2019/03/22/5c9455e9df280.png)

### 服务端异步获取数据
当服务端渲染的html文件数据需要通过请求另外的接口获取时，这个时候就需要服务端去请求数据，再将渲染完的 html 页面返回给客户端。

模拟接口返回数据。

**fetchData.js**
````javascript
export function getInitCount() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(10)
    }, 3000)
  })
}
````
服务端请求完数据之后，需要将给数据 React 组件使用。可以采用两种方式：

1. 通过 React 组件的 props 传入。
2. 通过 Router 的 context 传入，React 组件中再通过 props.staticContext 获取到。

**server.js**
````javascript
import { getInitCount } from './src/utils/fetchData'
//...
app.get('/*', (req, res) => {
  const renderedString = renderToString(
    <StaticRouter location={ req.url }>
      <App initCount={resData} />
    </StaticRouter>
  )
  //...
})
````
**App.js**
````javascript
class App extends Component {
  constructor(props) {
    super(props)
    this.state = {
      count: props.initCount || 0
    }
  }
  //...
}
````
这样可以看到服务器返回的 html 文件内容，已经将 count 的初始值 10 渲染进去了。

![服务端渲染](https://i.loli.net/2019/03/22/5c948522154ca.jpg)

但是此时页面显示的count值还是0，原因在于虽然使用了 hydrate() 方法，并不会将 html 结构重新渲染，但是 React 组件还是会走一遍生命周期流程，App 组件的 props.initCount 依旧是 0。

为了让客户端和服务端的props保持一致，需要将一个服务器生成的首屏 props 赋给客户端的全局变量。

修改下server.js 和 index.js
````javascript
app.get('/*', (req, res) => {
  getInitCount().then(resData => {
    const renderedString = renderToString(
        <StaticRouter location={req.url}>
          <App initCount={resData} />
        </StaticRouter>
      )
  
    fs.readFile(path.resolve('index.html'), 'utf8', (error, data) => {
      //...

      res.send(`${data
        .replace('<div id="app"></div>', `<div id="app">${renderedString}</div>`)
        .replace('</body>', `</body><script>window.__initCount__ = ${JSON.stringify(resData)}</script>`)}`)
    })
  })
})
````
````javascript
ReactDOM.hydrate(
    <BrowserRouter>
        <App initCount={window.__initCount__} />
    </BrowserRouter>
    , document.getElementById('app'))
````
注意 window.__initCount__ 需要在 app.js 前面加载。

![服务端渲染](https://i.loli.net/2019/03/22/5c9487b39b46b.png)

### 服务端无法支持图片、css等资源文件
如果代码中我们 import 了图片, svg, css 等非 js 资源，在客户端 webpack 的各种 loader 帮我们处理了这些资源，而 node 环境下只能识别 js。由于之前的例子没有引入额外的静态资源，所以没有出现这样的问题。

试着在 pageB 组件中引入图片。
````javascript
render() {
  return (
    <div>
      <div>B页面</div>
      <img src={require('../assets/img/hat.jpg')} alt=""/>
    </div>
  )
}
````

运行便报错了。

引入 webpack-isomorphic-tools 工具来解决这个问题。

webpack-isomorphic-tools 完成了两件事：

1. 以webpack插件的形式，预编译less（不局限于less，还支持图片文件、字体文件等），将其转换为一个 assets.json 文件保存到项目目录下。
2. require hook，所有less文件的引入，代理到生成的 JSON 文件中，匹配文件路径，返回一个预先编译好的 JSON 对象。

实现步骤：
1. 修改webpack.config.js

````javascript
var WebpackIsomorphicToolsPlugin = require('webpack-isomorphic-tools/plugin')

var webpackIsomorphicToolsPlugin = 
  // webpack-isomorphic-tools settings reside in a separate .js file 
  // (because they will be used in the web server code too).
  new WebpackIsomorphicToolsPlugin(require('./webpack-isomorphic-tools-configuration'))
  // also enter development mode since it's a development webpack configuration
  // (see below for explanation)
  .development()

// usual Webpack configuration
module.exports =
{
  context: '(required) your project path here',
  module:
  {
    loaders:
    [
      //...,
      {
        test: webpackIsomorphicToolsPlugin.regularExpression('images'),
        loader: 'url-loader?limit=1024', // any image below or equal to 10K will be converted to inline base64 instead
      }
    ]
  },

  plugins:
  [
    //...,
    webpackIsomorphicToolsPlugin
  ]
  //...
}
````
2. 添加 webpack-isomorphic-tools 配置文件 webpack-isomorphic-config.js

````javascript
module.exports = 
{
  assets:
  {
    images:
    {
      extensions: ['png', 'jpg', 'gif', 'ico', 'svg']
    }
  }
}
````

webpack-isomorphic-tools 启动时，会先等待指定目录下 assets.json 文件生成，只有该文件就绪后，require hook 才会进行，进而触发 server 回调，只有在此回调中执行的代码，才能保证进行了require hook。所以 server.js 文件变成了回调，需要新增额外的入口文件。

3. 增加mian.js

````javascript
var WebpackIsomorphicTools = require('webpack-isomorphic-tools');

global.webpackIsomorphicTools = new WebpackIsomorphicTools(require('./webpack-isomorphic-config'))
    .server('./', function () {
        //回调
        require('./server.js') //启动 server
  })
````
同时修改启动脚本
````javascript
"scripts": {
  "start": "npm run build && babel-node ./main.js --presets es2015,stage-0,react",
  "build": "webpack"
},
````
启动，会先生成 webpack-assets.json 文件, 通过映射关系可以正确处理非 js 文件。
````javascript
{
  "javascript": {
    "app": "app.js"
  },
  "styles": {},
  "assets": {
    "./src/assets/img/hat.jpg": "a80196c0daaadce1c8ef8446cc8212d5.jpg"
  },
  "webpack": {
    "version": "4.29.6"
  }
}
````
这样子页面也能正确加载图片了。

![服务端渲染](https://i.loli.net/2019/03/22/5c9495e4cd31b.png)

带来的问题: webpack-isomorphic-tools 这种 hook 方式，将整个Express Server置于自身的回调中，仿佛劫持了整个server，总不是显得那么的优雅。
## 源码地址

* [react-ssr](https://github.com/60kmlh/react-ssr)

## 参考资料

* [如何用 React 做服务端渲染](https://juejin.im/post/5c18c34ef265da615114ae78)
* [React16.x中的服务端渲染（SSR）](https://juejin.im/post/5b399412e51d4558dc4ae82d)
* [ReactDOMServer](https://zh-hans.reactjs.org/docs/react-dom-server.html)
* [react中出现的"hydrate"这个单词到底是什么意思?](https://www.zhihu.com/question/66068748)
* [webpack-isomorphic-tools](https://github.com/catamphetamine/webpack-isomorphic-tools)
