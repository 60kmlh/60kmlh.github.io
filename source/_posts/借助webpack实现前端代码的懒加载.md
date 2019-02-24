---
title: 借助webpack实现前端代码的懒加载
date: 2017-08-26
tags: ['webpack','性能优化']
categories: ['工具']
---
## 背景
随着前端项目的复杂度越来越大，要加载的js文件体积也越来越大。为了提高项目的首屏加载速度，实现在首屏时只加载首屏需要用到的js文件，用户使用其他功能时再加载对应的js文件，我们需要对js进行代码分割和懒加载。
## webppack的懒加载方案
### require.ensure()
webpack1(webpack2以上也可使用，但不推荐)使用require.ensure()定义一个代码切割点。
> Note: require.ensure only loads the modules, it doesn’t evaluate them.
````javascript
require.ensure(dependencies: String[], callback: function(require), errorCallback: function(error), chunkName: String)
````
dependencies定义异步加载模块的依赖的一些模块，会和加载的模块打包到一起。

callback定义回调函数，使用require参数可以在回调函数内动态引入其他模块。

errorCallback定义加载错误时的回调函数。

chunkName定义打包的chunk名称。
#### 加载js函数
````javascript
function fn2() {
  require.ensure([],function(){
    var utils = require('./utils')
    utils(function(){console.log('module2')})
  },'utils')
}

module.exports = fn2
````
#### 加载react组件
````javascript
loadTimeComponent() {
  if(this.state.timeComponent) {
    return true
  }
  var that = this
  require.ensure([],function(){
    var Time = require('./Time.js').default
    that.setState({
      timeComponent: <Time alertTime={(time) => that.alertTime(time)} />
    })
  },'Time')
}
````
#### 配合react-router3加载路由组件
````javascript
const Home = (location, cb) => {
  require.ensure([], require => {
    cb(null, require('./components/Home').default)
  }, 'Home')
}

function errorLoading(err) {
  console.error('Dynamic page loading failed', err);
}

class App extends Component {
  render() {
    return (
      <div>
        <Router history={hashHistory}>
          <Route path='/' getComponent={(Home)} />
        </Router>
      </div>
    )
  }
}
````
#### 配合react-router4加载路由组件
由于react-router4没有getComponent方法，需要借助bundle-loader
````javascript
import Bundle from './components/Bundle.js'
import HomeContainer from 'bundle-loader?lazy&name=[name]!./components/Home'

const Home = () => (
  <Bundle load={HomeContainer}>
    {(Home) => <Home />}
  </Bundle>
)
class App extends Component {
  render() {
    return (
      <div>
        <Route exact path='/' component={Home} />
      </div>
    )
  }
}

export default App

````
当Bundle组件的load方法被调用时，才会去加载对应的js文件。


### import()
webpack2 的ES2015 loader中提供了import()方法在运行时动态按需加载ES2015 Module。

webpack将import()看做一个分割点并将其请求的module打包为一个独立的chunk。import()以模块名称作为参数名并且返回一个Promise对象。

在Babel中使用import()方法，需要安装 dynamic-import插件并选择使用babel-preset-stage-3处理解析错误。

#### 加载js函数
```` javascript
function fn2() {
  import(/* webpackChunkName: "utils" */'./utils').then(module => {
    var utils = module //es6模块 module.default
    utils(function(){console.log('module2')})
  })
}

module.exports = fn2
// webpackChunkNam定义chunk名称
````
#### 加载react组件
配合异步组件asyncComponent实现，内部借助async和await实现异步加载。
react-router4中使用也是可以配合asyncComponent。
````javascript
import asyncComponent from './AsyncComponent'
const Head = asyncComponent(() => import(/* webpackChunkName: "Head" */'./Head'))

class App extends Component {
  render() {
    return (
      <div>
        <Head />
      </div>
    )
  }
} 

export default App
````
#### 配合react-router3加载路由
同样是借助getComponenet方法。
````javascript
<Route
  path='time' 
  getComponent={
    (location, cb) => import(/* webpackChunkName: "Time" */'./components/Time').
                      then(module => cb(null, module.default)).
                      catch(errorLoading)} 
/>
````
### bundle-loader和Bundle组件的实现原理
#### bundle-loader核心源码
````javascript
if(query.lazy) {
		result = [
			"module.exports = function(cb) {\n",
			"	require.ensure([], function(require) {\n",
			"		cb(require(", loaderUtils.stringifyRequest(this, "!!" + remainingRequest), "));\n",
			"	}" + chunkNameParam + ");\n",
			"}"];
	} else {
		result = [
			"var cbs = [], \n",
			"	data;\n",
			"module.exports = function(cb) {\n",
			"	if(cbs) cbs.push(cb);\n",
			"	else cb(data);\n",
			"}\n",
			"require.ensure([], function(require) {\n",
			"	data = require(", loaderUtils.stringifyRequest(this, "!!" + remainingRequest), ");\n",
			"	var callbacks = cbs;\n",
			"	cbs = null;\n",
			"	for(var i = 0, l = callbacks.length; i < l; i++) {\n",
			"		callbacks[i](data);\n",
			"	}\n",
			"}" + chunkNameParam + ");"];
	}
````
可以看到bundle-loader也是借助require.ensure()实现异步加载，而且对方法进行了封装，可以传参name和lazy，分别定义chunk的名称和是否懒加载。

#### Bundle组件的核心源码
````javascript
constructor(props) {
  super(props);

  this.state = {
      mod: null
  };
}

componentWillMount() {
    this.load(this.props)
}

componentWillReceiveProps(nextProps) {
    if (nextProps.load !== this.props.load) {
        this.load(nextProps)
    }
}

load(props) {
    this.setState({
        mod: null
    })
    props.load((mod) => {
        this.setState({
            // handle both es imports and cjs
            mod: mod.default ? mod.default : mod
        })
    })
}

render() {
    if(!this.state.mod)
        return false
    return this.props.children(this.state.mod)
}
````

当bundle-loader加载文件加载完成时，Bundle组件的componentWillReceiveProps钩子被触发，调用load方法设置加载到的组件，最后将组件返回。

### 完整代码

[源码](https://github.com/60kmlh/codeSplitting)

### 参考资料
[webpack1 code-splitting](http://webpack.github.io/docs/code-splitting.html)

[webpack1 lazy-loading](https://webpack.js.org/guides/lazy-loading/)

[基于webpack Code Splitting实现react组件的按需加载](https://yq.aliyun.com/articles/71200)
