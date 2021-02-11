# React 性能优化之组件动态加载（react-loadable）

## 什么是react-loadable
npm desc：
A higher order component for loading components with dynamic imports.
一个异步加载component的高阶组件
React 项目打包时，如果不进行异步组件的处理，那么所有页面所需要的 js 都在同一文件中（bundle.js），整个js文件很大，从而导致首屏加载时间过长。
所有，可以对组件进行异步加载处理，通常可以使用 React-loadable。

### 组件拆分
做路由拆分可以达到一定程度的性能优化，loadable本质上做的事组件拆分


### 基本用法
```javascript
import Loadable from 'react-loadable';
 
const LoadableBar = Loadable({
  loader: () => import('./components/Bar'),
  loading() {
    return <div>Loading...</div>
  }
});

 
class MyComponent extends React.Component {
  render() {
    return <LoadableBar/>;
  }
}
```

### 简易实现
基本思路，在组件和页面之间加一层代理，通过文件异步加载实现懒加载

```javascript
import React, { Component } from "react";

const loadable = ({ loader, loading: Loading }) => {
  return class loadableComponent extends Component {
    state = {
      LoadedComponent: null
    };
    componentDidMount() {
      //loader是一个函数 执行import操作
      loader().then(res => {
        console.log(res.default);
        this.setState({ LoadedComponent: res.default });
      });
    }
    render() {
      const { LoadedComponent } = this.state;
      return LoadedComponent ? <loadedComponent /> : <Loading />;
    }
  };
};
export default loadable;
```

## 源码分析



### 源码结构

这是loadable的源码结构

 ![image](http://cdn.blog.yangzhedi.com/loadable/loadable.png)

最后export出来的是一个`Loadable`函数，所以我们从`Loadable`开始分析。

```javascript
module.exports = Loadable;
function Loadable(opts) {
  return createLoadableComponent(load, opts);
}
```

Loadable接受一个参数`opts`,再调用了`createLoadableComponent`函数，传入了load函数和opts。



#### load函数

```javascript
function load(loader) {
    let promise = loader();
    let state = {
        loading: true,

        loaded: null,
        error: null
    };
    state.promise = promise.then(loaded => {

        state.loading = false;
        state.loaded = loaded;
        return loaded;
    }).catch(err => {

        state.loading = false;
        state.error = err;
        throw err;
    });
    return state;
}
```

这里`load`又是一个函数，接受一个`loader`参数。我们先放在这里，一会在回来看这个`loader`。



#### createLoadableComponent

![createLoadableComponent](http://cdn.blog.yangzhedi.com/loadable/createLoadableComponent.png)

这个函数是整个库的毕竟重要的函数了。（因为本文是浅析，所以先只分析主逻辑的函数，别的健壮性支持函数可能会之后再分析）



##### if(!options.loading){...}

`options.loading`就是上文提到的一个无状态组件,负责显示"Loading中"，如果不存在，会报错，需要一个Loading中显示的组件。



##### let opts = Object.assign(...,options)

```javascript
let opts = Object.assign({
        loader: null,
        loading: null,
        delay: 200,
        timeout: null,
        render: render,
        webpack: null,
        modules: null,
    }, options);
```

就是初始化一下opts的值，赋给未传入参数初始值，防止接下来的判断报错。



##### function init() {...}

```javascript
function init() {
    if (!res) {
        res = loadFn(opts.loader);
    }
    return res.promise;
}
```

`init`之前声明过一个`res`，`init`就是为`res`赋值。在`init`里调用了`loadFn`,就是上文说过的`load函数`。

load函数接收一个参数，就是之前`() => import('./my-component')`，这里的`import()`方法，返回一个Promise对象，`[[PromiseValue]].default`就是待load组件的function。



 ![image](http://cdn.blog.yangzhedi.com/loadable/import.png)



load函数里初始化了一个state对象，并在import()方法返回的Promise中，对state的属性赋值，`loading`代表是否加载完成， `loaded`为加载的对象，这里的`state.loaded`已经是`[[PromiseValue]]`了。



##### return class LoadableComponent extends React.Component ...

因为react loadable的描述终究是一个高阶组件，如果对高阶组件不了解的，可以去看 [深入React高阶组件(HOC)](https://juejin.im/post/5adddc57f265da0b8635de56)这篇文章。 所以`createLoadableComponent`最后返回的是一个React Component。

一开始的`constructor`里，调用了init，将res的几个属性，分别赋值给this.state作为组件初始化。

```javascript
this.state = {
    error: res.error,
    pastDelay: false,
    timedOut: false,
    loading: res.loading,
    loaded: res.loaded
};
```

之后在`componentWillMount`中，做了这些操作

- 设置了个标记位`this._mounted`，默认为true
- 进行一些判断，如果`res.loading`为true，说明之前的init执行出错，直接return；
- 如果`opts.delay`和`opts.timeout`有值，且为number属性的话，就加个定时器。
- 声明update函数，如果`this._mounted`为false，重新将res的属性更新到state里。

`render`中进行了判断，

- 如果`this.state.loading`或者`this.state.error`为true，就是整体状态是正在加载ing或者出错了，就用`React.createElement`生成出loading的过渡组件，
- 如果`this.state.loaded`有值了，说明传入的`loader`的promise异步操作执行完成，就开始渲染真正的组件，调用`opts.render`方法.(此时的this.state.loaded就是`[[PromiseValue]]`) 

![image](http://cdn.blog.yangzhedi.com/loadable/import2.png)



```javascript
function resolve(obj) {
    return obj && obj.__esModule ? obj.default : obj;
}
function render(loaded, props) {
    return React.createElement(resolve(loaded), props);
}
```

render的时候进行判断，当__esModule为true的时候，标识module为es模块，那么obj默认返回obj.default，那么obj默认返回obj。之后再调用`React.createElement`进行渲染。

至于props是什么？在项目里打印发现，就是原组件的props。所以render出的，就和正常在代码中写的React Component是一样的。 

![image](http://cdn.blog.yangzhedi.com/loadable/props.png) 



然后，整个loadable就完成了。

## 总结

这个库的设计还是非常巧妙，利用import返回一个promise对象的性质，进行loading的异步操作，有点像图片懒加载的原理。下面是划重点系列，

- import() 可以返回一个promise对象
- React.createElement() API
- Promise对象原理。
