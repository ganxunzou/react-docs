# React-Router 4
单页面应用页面跳转，前进后退单靠页面切换实现起来很麻烦。于是就有了 `React-Router` ：前段静态页面中的路由。它通过管理 URL，实现组件的切换和状态的变化。
`React-Router 4` 之后没有了集中路由的概念，所有的路由都是 `react` 的组件。后面在详细哦 `React-Router` 升级。

`React-Router 4` 之后将 `react-router` 拆分成两个库：`react-router` 和 `react-router-dom`

- `react-router` API
```JS
export MemoryRouter from './MemoryRouter'
export Prompt from './Prompt'
export Redirect from './Redirect'
export Route from './Route'
export Router from './Router'
export StaticRouter from './StaticRouter'
export Switch from './Switch'
export generatePath from './generatePath'
export matchPath from './matchPath'
export withRouter from './withRouter'

```

- `react-router-dom` API
```JS
export BrowserRouter from './BrowserRouter'
export HashRouter from './HashRouter'
export Link from './Link'
export MemoryRouter from './MemoryRouter'
export NavLink from './NavLink'
export Prompt from './Prompt'
export Redirect from './Redirect'
export Route from './Route'
export Router from './Router'
export StaticRouter from './StaticRouter'
export Switch from './Switch'
export generatePath from './generatePath'
export matchPath from './matchPath'
export withRouter from './withRouter'
```

这里看到有些导出的API是相同的，比如 `MemoryRouter` 。其实他们是一样的。 `react-router-dom` 是将 `react-router` 中的API重新在导出了一遍。
```JS
// Written in this round about way for babel-transform-imports
import { MemoryRouter } from 'react-router'
export default MemoryRouter

```
另外 `react-router-dom` 新增了以下几个增强的API。所以我们在开发中调用的时候，所有的接口调用可以直接通过 `react-router-dom` 导入。
```JS
export BrowserRouter from './BrowserRouter'
export HashRouter from './HashRouter'
export Link from './Link'
export NavLink from './NavLink'
```

# Router
我们先来讲讲根路由。一切发生都是从根开始。
```JS
class Router extends React.Component {}
```
是不是很熟悉，对没错就是一个 `react` 组件。所以再次提醒自己 `react-router 4` 一切都是组件。 `Router` 是一个顶层组件，在组件的 `context` 中注入一个 `router` 属性，并订阅 `history` 的变化，然后精确匹配 `/` 路径。特别需要注意的是 `Router` 的子组件只能放置一个，如果想放置多个子组件的话，需要外层用一个容器包裹起来。下面的核心代码。

- `context` 注入 `router` 属性
```JS
// 提供子组件从`context`获取`router`
getChildContext() {
    return {
      router: {
        ...this.context.router,
        history: this.props.history,
        route: {
          location: this.props.history.location, // 因为是根节点，location 直接从 history 获取
          match: this.state.match
        }
      }
    }
  }
// 声明router类型
static childContextTypes = {
    router: PropTypes.object.isRequired
}
```

- 监听 `history` 的改变
```JS
// 监听改变
this.unlisten = history.listen(() => {
    this.setState({
    match: this.computeMatch(history.location.pathname)
    })
})
// 精确匹配
computeMatch(pathname) {
    return {
      path: '/',
      url: '/',
      params: {},
      isExact: pathname === '/'
    }
  }
```

- 精确匹配 `/` 路径
```JS
state = {
  match: this.computeMatch(this.props.history.location.pathname)
}

computeMatch(pathname) {
  return {
    path: '/',
    url: '/',
    params: {},
    isExact: pathname === '/' //精确匹配
  }
}
```

# Route
```JS
class Route extends React.Component
```
`Route` 将 `URL` 地址和页面绑定的组件。也是一个正规的 `react` 组件。

`Route` 组件提供了以下属性
```JS
static propTypes = {
    computedMatch: PropTypes.object, // private, from <Switch>
    path: PropTypes.string, // 路径
    exact: PropTypes.bool,  // 是否精确匹配
    strict: PropTypes.bool, // 是否严格模式
    sensitive: PropTypes.bool,
    component: PropTypes.func, // 组件
    render: PropTypes.func,    // 
    children: PropTypes.oneOfType([
      PropTypes.func,
      PropTypes.node
    ]), // 子组件 必须只能有一个
    location: PropTypes.object // 地址
  }
```
`Route`组件的功能是`path` 和组件绑定（组件多种定义（优先级从左导右）：`component`,`render`，`children`）。它的实现原理是将 `path` 与 `history`的`location` 或上层`route`的`location`进行匹配，匹配成功然后渲染组件。

`Route` 和`Router`最大的差别在于`Route` 重置了 `context` 中`router`属性。简单对比一下两个源码：
- `Router`
```JS
getChildContext() {
  return {
    router: {
      ...this.context.router,
      history: this.props.history,
      route: {
        location: this.props.history.location, //直接从history中获取location
        match: this.state.match
      }
    }
  }
}
```
- `Route`
```JS
getChildContext() {
    return {
      router: {
        ...this.context.router,
        route: {
          location: this.props.location || this.context.router.route.location,// 从上层route中获取location
          match: this.state.match
        }
      }
    }
  }
```
这个就是为什么`Route`可以多层级嵌套使用的原因了。





