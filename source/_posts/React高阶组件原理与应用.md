---
title: React高阶组件的原理与应用
date: 2018-09-22
tags: ['react', 'hoc']
categories: ['react']
---
## 什么是高阶组件
* 高阶函数

如果一个函数 接受一个或多个函数作为参数或者返回一个函数 就可称之为 高阶函数。
<!--more-->
* 高阶组件

如果一个函数 接受一个或多个组件作为参数并且返回一个组件 就可称之为 高阶组件。

当高阶组件中返回的组件是 无状态组件（Stateless Component） 时，该高阶组件其实就是一个 高阶函数，因为 无状态组件 本身就是一个纯函数。
## 高阶组件的实现
**React 中的高阶组件主要有两种形式：属性代理 和 反向继承。**
### 属性代理（Props Proxy）
一个函数接受一个 WrappedComponent 组件作为参数传入，并返回一个继承了 React.Component 组件的类，且在该类的 render() 方法中返回被传入的 WrappedComponent 组件。

此时，能对 render() 方法里的 WrappedComponent 组件进行一些处理：
* 操作 props
````javascript
function HigherOrderComponent(WrappedComponent) {
  return class extends React.Component {
    render() {
      const newProps = {
        value: 'defalut'
      };
      return <WrappedComponent {...this.props} {...newProps} />;
    }
  };
}
````
* 抽离 state
利用 props 和回调函数把 state 抽离出来：

````javascript
function withOnChange(WrappedComponent) {
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        value: '',
      };
    }
    onChange = (value) => {
      this.setState({
        value
      });
    }
    render() {
      const newProps = {
        value: this.state.value,
        onChange: this.onChange,
      };
      return <WrappedComponent {...this.props} {...newProps} />;
    }
  };
}
````
* 通过 ref 访问到组件实例

在需要访问 DOM element （使用第三方 DOM 操作库）的时候就会用到组件的 ref 属性。它只能声明在 Class 类型的组件上，而无法声明在函数（无状态）类型的组件上。

不能在无状态组件（函数类型组件）上使用 ref 属性，因为无状态组件没有实例。
ref 的值可以是字符串（不推荐使用）也可以是一个回调函数，如果是回调函数的话，它的执行时机是：

* 组件被挂载后（componentDidMount），回调函数立即执行，回调函数的参数为该组件的实例。
* 组件被卸载（componentDidUnmount）或者原有的 ref 属性本身发生变化的时候，此时回调函数也会立即执行，且回调函数的参数为 null。

````javascript
function HigherOrderComponent(WrappedComponent) {
  return class extends React.Component {
    executeInstanceMethod = (wrappedComponentInstance) => {
      wrappedComponentInstance.someMethod();
    }
    render() {
      return <WrappedComponent {...this.props} ref={this.executeInstanceMethod} />;
    }
  };
}
````
* 用其他元素包裹传入的组件 WrappedComponent

给 WrappedComponent 组件包一层背景色为 #fafafa 的 div 元素：

````javascript
function withBackgroundColor(WrappedComponent) {
  return class extends React.Component {
    render() {
      return (
        <div style={{ backgroundColor: '#fafafa' }}>
            <WrappedComponent {...this.props} />
        </div>
      );
    }
  };
}
````
### 反向继承（Inheritance Inversion）
一个函数接受一个 WrappedComponent 组件作为参数传入，并返回一个继承了该传入 WrappedComponent 组件的类，且在该类的 render() 方法中返回 super.render() 方法。

作用：
* 操作 state

高阶组件中可以读取、编辑和删除 WrappedComponent 组件实例中的 state。甚至可以增加更多的 state 项，但是 非常不建议这么做 因为这可能会导致 state 难以维护及管理。

````javascript
function withLogging(WrappedComponent) {
  return class extends WrappedComponent {
    render() {
      return (
        <div>
          <h2>Debugger Component Logging...</h2>
          <p>state:</p>
          <pre>{JSON.stringify(this.state, null, 4)}</pre>
          <p>props:</p>
          <pre>{JSON.stringify(this.props, null, 4)}</pre>
          {super.render()}
        </div>
      );
    }
  };
}
````
* 渲染劫持（Render Highjacking）

渲染劫持 是因为高阶组件控制着 WrappedComponent 组件的渲染输出，通过渲染劫持我们可以：

* 有条件地展示元素树（element tree）
* 操作由 render() 输出的 React 元素树
* 在任何由 render() 输出的 React 元素中操作 props
* 用其他元素包裹传入的组件 WrappedComponent （同 属性代理）

````javascript
function withLoading(WrappedComponent) {
    return class extends WrappedComponent {
        render() {
            if(this.props.isLoading) {
                return <Loading />;
            } else {
                return super.render();
            }
        }
    };
}
````

````javascript
function HigherOrderComponent(WrappedComponent) {
    return class extends WrappedComponent {
        render() {
            const tree = super.render();
            const newProps = {};
            if (tree && tree.type === 'input') {
                newProps.value = 'something here';
            }
            const props = {
                ...tree.props,
                ...newProps,
            };
            const newTree = React.cloneElement(tree, props, tree.props.children);
            return newTree;
        }
    };
}
````
## 高阶组件存在的问题
* 静态方法丢失
因为原始组件被包裹于一个容器组件内，也就意味着新组件会没有原始组件的任何静态方法。

须将静态方法做拷贝：
````javascript
function HigherOrderComponent(WrappedComponent) {
  class Enhance extends React.Component {}
  // 必须得知道要拷贝的方法
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
````
借助库 hoist-non-react-statics 来自动处理，它会 自动拷贝所有非 React 的静态方法。
````javascript
import hoistNonReactStatic from 'hoist-non-react-statics';

function HigherOrderComponent(WrappedComponent) {
  class Enhance extends React.Component {}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
````
* refs 属性不能透传
高阶组件可以传递所有的 props 给包裹的组件 WrappedComponent，但是有一种属性不能传递，它就是 ref。与其他属性不同的地方在于 React 对其进行了特殊的处理。

如果你向一个由高阶组件创建的组件的元素添加 ref 引用，那么 ref 指向的是最外层容器组件实例的，而不是被包裹的 WrappedComponent 组件。

React 为我们提供了一个名为 React.forwardRef 的 API 来解决这一问题（在 React 16.3 版本中被添加）：
````javascript
function withLogging(WrappedComponent) {
  class Enhance extends WrappedComponent {
    componentWillReceiveProps() {
      console.log('Current props', this.props);
      console.log('Next props', nextProps);
    }
    render() {
      const {forwardedRef, ...rest} = this.props;
      // 把 forwardedRef 赋值给 ref
      return <WrappedComponent {...rest} ref={forwardedRef} />;
    }
  };

  // React.forwardRef 方法会传入 props 和 ref 两个参数给其回调函数
  // 所以这边的 ref 是由 React.forwardRef 提供的
  function forwardRef(props, ref) {
    return <Enhance {...props} forwardRef={ref} />
  }

  return React.forwardRef(forwardRef);
}
const EnhancedComponent = withLogging(SomeComponent);
````
* 反向继承不能保证完整的子组件树被解析
如果渲染 elements tree 中包含了 function 类型的组件的话，这时候就不能操作组件的子组件了。
## 遵循的约定
* props 保持一致
保持原有组件的 props 不受影响。
* 不要以任何方式改变原始组件 WrappedComponent
对原有组件产生了副作用，失去了组件复用的意义，所以请通过 纯函数（相同的输入总有相同的输出） 返回新的组件。
* 透传不相关 props 属性给被包裹的组件 WrappedComponent
* 不要再 render() 方法中使用高阶组件
调用高阶函数的时候每次都会返回一个新的组件，每次 render 的时候，都会使子对象树完全被卸载和重新渲染，重新加载一个组件会引起原有组件的状态和它的所有子组件丢失。
* 使用 compose 组合高阶组件
可以显著提高代码的可读性和逻辑的清晰度。
````javascript
// 不要这么使用
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))；
// 可以使用一个 compose 函数组合这些高阶组件
// lodash, redux, ramda 等第三方库都提供了类似 `compose` 功能的函数
const enhance = compose(withRouter, connect(commentSelector))；
const EnhancedComponent = enhance(WrappedComponent)；
````
* 包装显示名字以便于调试
高阶组件创建的容器组件在 React Developer Tools 中的表现和其它的普通组件是一样的。为了便于调试，可以选择一个显示名字，传达它是一个高阶组件的结果。
````javascript
const getDisplayName = WrappedComponent => WrappedComponent.displayName || WrappedComponent.name || 'Component';
function HigherOrderComponent(WrappedComponent) {
  class HigherOrderComponent extends React.Component {/* ... */}
  HigherOrderComponent.displayName = `HigherOrderComponent(${getDisplayName(WrappedComponent)})`;
  return HigherOrderComponent;
}
````
## 常见应用场景
* 权限控制
利用高阶组件的 条件渲染 特性可以对页面进行权限控制，权限控制一般分为两个维度：页面级别 和 页面元素级别。
* 组件渲染性能追踪
根据react父子组件的渲染顺序和父组件子组件生命周期规则捕获子组件的生命周期，可以方便的对某个组件的渲染时间进行记录：
````javascript
class Home extends React.Component {
    render() {
        return (<h1>Hello World.</h1>);
    }
}
function withTiming(WrappedComponent) {
  return class extends WrappedComponent {
    constructor(props) {
      super(props);
      this.start = 0;
      this.end = 0;
    }
    componentWillMount() {
      super.componentWillMount && super.componentWillMount();
      this.start = Date.now();
    }
    componentDidMount() {
      super.componentDidMount && super.componentDidMount();
      this.end = Date.now();
      console.log(`${WrappedComponent.name} 组件渲染时间为 ${this.end - this.start} ms`);
    }
    render() {
      return super.render();
    }
  };
}

export default withTiming(Home);
````
* 页面复用
将多个页面不同的部分抽离到外部传入，从而实现页面的复用。
## Function as Child Components
另一种类似高阶组件的方式叫做 Function as Child Components。它的思路是将函数（执行结果是返回新的组件）作为子组件传入，在父组件的render方法中执行此函数，可以传入特定的参数作为子组件的props。
````javascript
class StudentWithAge extends React.Component {
  componentWillMount() {
    this.setState({
      name: '小红',
      age: 25,
    });
  }
  render() {
    return (
      <div>
        {this.props.children(this.state.name, this.state.age)}
      </div>
    );
  }
}
````
使用：
````javascript
<StudentWithAge>
  {
    (name, age) => {
      let studentName = name;
      if (age > 22) {
          studentName = `大学毕业的${studentName}`;
      }
      return <Student name={studentName} />;
    }
  }
</StudentWithAge>
````

相比高阶组件的优点：
1. 代码结构上少掉了一层（返回高阶组件的）函数封装。
2.调试时组件结构更加清晰
3.从组件复用角度来看，父组件和子组件之间通过children连接，两个组件其实又完全可以单独使用，内部耦合较小。当然单独使用意义并不大，而且高阶组件也可以通过组合两个组件来做到。
缺点：
1. （返回子组件）函数占用了父组件原本的props.children；
2. （返回子组件）函数只能进行调用，无法劫持劫持原组件生命周期方法或取到static方法；
3. （返回子组件）函数作为子组件包裹在父组件中的方式看起来虽灵活但不够优雅；
4. 由于子组件的渲染控制完全通过在父组件render方法中调用（返回子组件）函数，无法通过shouldComponentUpdate来做性能优化。

所以这两种方式各有优劣，可根据具体场景选择。
## 参考资料
* [Higher-Order Components - React](https://reactjs.org/docs/higher-order-components.html)
* [React 中的高阶组件及其应用场景](https://juejin.im/post/5c72b97de51d4545c66f75d5)
* [React高阶组件实践](https://juejin.im/post/59b36b416fb9a00a636a207e)