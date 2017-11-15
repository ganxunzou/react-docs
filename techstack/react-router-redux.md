# react-router-redux
react-router-redux 提供了以下几个API
```JS
export ConnectedRouter from './ConnectedRouter'
export { getLocation, createMatchSelector } from './selectors'
export { LOCATION_CHANGE, routerReducer } from './reducer'
export {
  CALL_HISTORY_METHOD,
  push, replace, go, goBack, goForward,
  routerActions
} from './actions'
export routerMiddleware from './middleware'

```

# ConnectedRouter
ConnectedRouter 从名字看是连接到Router，把在根Router后面做一层包装，主要的作用是订阅Router的`location`的变化，`location`变化会派发一个`LOCATION_CHANGE`的`action`。

订阅 history 即 `location`
```JS
componentDidMount() {
    const { history } = this.props
    this.unsubscribeFromHistory = history.listen(this.handleLocationChange)
  }
```

location 变化派发一个`LOCATION_CHANGE`的`action`
```JS
 handleLocationChange = location => {
    this.store.dispatch({
      type: LOCATION_CHANGE,
      payload: location
    })
  }

  componentWillMount() {
    const { store:propsStore, history } = this.props
    this.store = propsStore || this.context.store
    this.handleLocationChange(history.location)
  }
```

# getLocation
获取当前`state`中存的`location`
```JS
export const getLocation = state => state.router.location
```
# createMatchSelector
创建一个匹配选择器。就是根据传入的`path`与当前`state`中的路径`pathname`（注意不是`location`）进行匹配，匹配成功返回`router`的`Macth`结构对象（结构：`{props,pathname}`）,如果匹配失败返回最后一次匹配成功的`Match`

# LOCATION_CHANGE
一个常量，用作`location`改变时派发`action`的`type`属性值
```JS
export const LOCATION_CHANGE = '@@router/LOCATION_CHANGE'
```

# routerReducer
首先从名字上看就知道，它是一个`reducer`。他的功能是在`Router`改变的时候生成新的`state`。
```JS
export function routerReducer(state = initialState, { type, payload } = {}) {
  if (type === LOCATION_CHANGE) {
    return { ...state, location: payload }
  }

  return state
}
```

# CALL_HISTORY_METHOD
一个常量，调用`history`操作是派发`action`的`type`值。`history`操作一般有
- `push` ： 新增一个路径
- `replace`：替换现有的路径，没有堆栈所以不能回退。
- `go`：跳转到指定路径
- `goBack`：回退
- `goForward` ： 前进

```JS
export const CALL_HISTORY_METHOD = '@@router/CALL_HISTORY_METHOD'

function updateLocation(method) {
  return (...args) => ({
    type: CALL_HISTORY_METHOD,
    payload: { method, args }
  })
}

/**
 * These actions correspond to the history API.
 * The associated routerMiddleware will capture these events before they get to
 * your reducer and reissue them as the matching function on your history.
 */
export const push = updateLocation('push')
export const replace = updateLocation('replace')
export const go = updateLocation('go')
export const goBack = updateLocation('goBack')
export const goForward = updateLocation('goForward')

export const routerActions = { push, replace, go, goBack, goForward }

```

# routerMiddleware
路由中间件。将注入的`histroy`和对外提供的操作`hisotry`的接口关联。前面我们说到有`push`，`replace`等很多操作`history`的API，都是通过这个中间件来操作`history`对象。
```JS
import { CALL_HISTORY_METHOD } from './actions'

/**
 * This middleware captures CALL_HISTORY_METHOD actions to redirect to the
 * provided history object. This will prevent these actions from reaching your
 * reducer or any middleware that comes after this one.
 */
export default function routerMiddleware(history) {
  return () => next => action => {
    if (action.type !== CALL_HISTORY_METHOD) {
      return next(action)
    }

    const { payload: { method, args } } = action
    history[method](...args)
  }
}
```

ES6代码理解起来，还是有些吃力。转成ES5代码后：
```JS
function routerMiddleware(history) {
  return function () {
    return function (next) {
      return function (action) {
        // 不是操纵历史，action继续下发
        if (action.type !== CALL_HISTORY_METHOD) {
          return next(action);
        }

        var _action$payload = action.payload,
            method = _action$payload.method,
            args = _action$payload.args;

        history[method].apply(history, _toConsumableArray(args));
      };
    };
  };
}
```