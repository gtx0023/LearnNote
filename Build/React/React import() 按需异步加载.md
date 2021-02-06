import()方法返回的是一个Promise，Promise的返回值只能通过then()来读取，所以你会发现官方的3种使用场景全都是在then()里面操作。

asyncComponent.jsx

```javascript
import React, { Component } from 'react';
 
const asyncComponent = (importComponent) => {
  return class extends Component {
    constructor() {
      super();
      this.state = {
        component: null
      }
    }
    componentDidMount() {
      importComponent()
        .then(cmp => {
          this.setState({ component: cmp.default });
        });
    }
    render() {
      const C = this.state.component;
      return C ? <C {...this.props} /> : null;
    }
  }
};
 
export default asyncComponent;
```

app.jsx
```javascript
import React, { Component } from 'react';
// 路由依赖
import { HashRouter, Route, Switch } from 'react-router-dom';
// 异步组件
import AsyncComponent from './asyncComponent.jsx';
// 组件页面
const Home = AsyncComponent(() => import(/* webpackChunkName: "Home" */ './Home/index.jsx'));
const City = AsyncComponent(() => import(/* webpackChunkName: "City" */ './City/index.jsx'));
const Search = AsyncComponent(() => import(/* webpackChunkName: "Search" */ './Search/index.jsx'));
const User = AsyncComponent(() => import(/* webpackChunkName: "User" */ './User/index.jsx'));
const NotFound = AsyncComponent(() => import(/* webpackChunkName: "NotFound" */ './NotFound/index.jsx'));
const Detail = AsyncComponent(() => import(/* webpackChunkName: "Detail" */ './Detail/index.jsx'));
const Login = AsyncComponent(() => import(/* webpackChunkName: "Login" */ './Login/index.jsx'));
// 全局样式
import '@/static/css/common.less';
import '@/static/css/font.css';
 
class App extends Component {
  constructor() {
    super();
  }
  render() {
    return (
      <HashRouter>
        <Switch>
          <Route path="/" exact component={Home} />
          <Route path="/city" component={City} />
          <Route path="/Search/:category/:keyword?" component={Search} />
          <Route path="/User" component={User} />
          <Route path="/Detail/:id" component={Detail} />
          <Route path="/login/:router?" component={Login} />
          <Route path="*" component={NotFound} />
        </Switch>
      </HashRouter>
    );
  }
}
 
export default App;
```
