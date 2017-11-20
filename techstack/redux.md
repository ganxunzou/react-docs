# Redux
redux 是一个状态`state`管理器，`state`和视图是一一对应的。即通过视图我能知道对应的`state`，通过`state`也能知道对应的视图。

在react中控制视图刷新的有两个属性：`state`和`props`。`redux`在`react`中实现`state`调整对应视图刷新，是用到`react`中`props`。这里还是需要简单的区别一下。

`redux`采用发布订阅的方式将`state`和`view`连接到一起。`redux`提供了`subscribe`函数进行订阅`state`的调整。一般我们会在根视图做这个监听。
```JS
const render = (Component) => {
  ReactDOM.render(
    <AppContainer>
      <Component store={store} />
    </AppContainer>,
    document.getElementById('app'),
  );
};

render(App);
//订阅，state修改重新刷新视图
store.subscribe(() => {
	render(App);
});

```
## Redux API
redux提供了四个API，分别是：
- createStore
- combineReducers
- applyMiddleware
- bindActionCreators

基础应用最常用的两个API：`createStore` 和 `combineReducers`。我们接下来一一说明一下各个API。

## createStore
```
var store = createStore(reducer)
```
`createStore`携带一个`reducer`参数并返回`store`的函数。这个函数一般只能调用一次。因为`redux`要求`store`全局只能有一个。这里引申出了两个API：`store`和`reducer`。

### reducer
```JS
reducer = function(state,action){return newState}
```
`reducer`其实是一个携带`state`和`action`参数的纯函数。他的作用是根据老的`state`和新的`action`返回一个新的`state`。简单理解就是`reducer`是负责生产`state`的函数。

简单`reducer`函数示例:
```JS
export default (state = [] , action) => {
    switch (action.type) {
        case 'ADD_TODO':
            return [
                ...state,
                {
                    id: action.id,
                    text: action.text,
                    completed: action.completed,
                },
            ];
        default:
            return state;
    }
};
```

### store 
`redux`要求`store`必须只能是一个，可以把它类比成数据库实例。`store`提供了有以下几个API: ` dispatch` ， `getState` ， ` subscribe`

 - `dispatch(action)`

`dispatch(action)`功能是派发一个`action`（PS:action后面会详细讲，这里知道他是包含`type`属性的简单对象，类似：`{type:"TODO"}`）。

- `getState()`

`getState()`是一个无参函数，返回当前的`state`。这里可以类比一下`react`中的`state`。`reac`体重`state`改变会导致视图刷新，同理`getState`返回的新值，也会导致页面刷新。`redux`中的设计思想是：`state`和`view`是一一对应的。即知道`view`能猜到对应的`state`，知道`state`我也能猜到对应的`view`。

- `subscribe(func)`

`subscribe`从名字看就知道是订阅的意思，那他订阅什么呢？前面我们说到：`dispatch(action) -> reducer -> new state -> getState()`这么一个过程。这里面是不是少了点什么？，说好的`state`和`view`一一对应，`view`呢？这就是`subscribe`的作用了。

`subscribe`监听所有的`dispatch`操纵。调用`dispatch(action)`操作后，所有需要获取新数据的地方（一般在render中），都需要通过`getState`函数获取。
```JS
 /**
   * Adds a change listener. It will be called any time an action is dispatched,
   * and some part of the state tree may potentially have changed. You may then
   * call `getState()` to read the current state tree inside the callback.
   */
```
`subscribe`函数惨一个`function`参数，一般情况下在根组件中处理这个监听。在监听处理函数中刷新根组件。代码示例如下：
```JS
import React from 'react';
import ReactDOM from 'react-dom';
import { AppContainer } from 'react-hot-loader';

import App from './App';
import store from './store';

const render = (Component) => {
  ReactDOM.render(
    <AppContainer>
      <Component store={store} />
    </AppContainer>,
    document.getElementById('app'),
  );
};

render(App);

// Webpack Hot Module Replacement API
if (module.hot) {
  module.hot.accept('./App', () => { render(App); });
}

store.subscribe(() => {
	render(App);
});

```
那每次都从根组件渲染，会不会影响性能和组件刷新界面闪烁问题呢？因为`react`采用的是`diff`算法，所以性能问题应该不太会有，界面刷新走了也是`react`的`props`改变页面刷新的逻辑，所以理论上不会闪烁。

## combineReducers
前面讲到`createStore(reducer)`传入reducer参数，reducer是一个纯函数返回新的state。代码类似下面：
```JS
export default (state = [] , action) => {
    switch (action.type) {
        case 'ADD_TODO':
            return [
                ...state,
                {
                    id: action.id,
                    text: action.text,
                    completed: action.completed,
                },
            ];
        default:
            return state;
    }
};
```
如果一个复杂的前段引用，这么写`reducer`会有无数的`swicth`分支，`reducer`就会变大巨大和臃肿，这肯定不是我们想要的。所以`redux`提供了`combineReducer`函数，从名字上就能看出他的意思：组合`reducer`。可以看下面代码示例：
```JS
import { combineReducers } from 'redux';

import counter from './counter';
import todo from './todo';

export default combineReducers({
    counter, //特别注意这是 class properties写法，还可以写成："counter":counter。
    todo,
});
```
当然你还可以在每个独立的`reducer`中继续拆分。例如在`todo reducer`中在用`combineReducer`组合,然后 `export` 出组合后的的`reducer`


## applyMiddleware 
```JS
import { createStore } from 'redux';
import reducers from '../reducers';
import thunk from 'redux-thunk'

const store = createStore(
	reducers,
  applyMiddleware(thunk), //支持传多个
);

export default store;
```
`redux`提供了中间件支持，用于完善`redux`复杂应用中的使用。比如我们这里抛出一个`Async Action`概念（后面讲）。

先解释一下中间件（PS:个人理解）：一般就是衔接两个应用（or服务），并提供一些增强的功能。比如常说的：日志中间件、操作统计中间件、请求拦截中间件等。

现在我们知道`applyMiddleware`的作用，就是负责增强`redux`功能。那增强`redux`什么功能？又是怎么增强的呢？看源码把：
```JS
export default function applyMiddleware(...middlewares) {
  return (createStore) => (...args) => {
    const store = createStore(...args)
    let dispatch = store.dispatch
    let chain = []

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
分解代码：
- 所有的中间件传递一个包含：getState和dispatch的简单对象参数
```
const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
```
- 合并所有的中间件，重构成新的`dispatch`
```JS
dispatch = compose(...chain)(store.dispatch)
return {
      ...store,
      dispatch
    }
```
那这就清楚了，`redux`的中间件是增强`dispatch`的。怎么增强呢：多次包装（PS:要清楚一点，dispatch也是一个纯函数）。就是先执行中间件的`dispatch`函数，最后执行`redux`的`dispatch`函数。就是这么简单粗暴的理解它。


## bindActionCreators
```JS
function bindActionCreator(actionCreator, dispatch) {
  return function() { return dispatch(actionCreator.apply(this, arguments)) }
}
```
这个函数so easy。正常我们的写法如下：
```JS
var action = ActionCreator();
dispatch(action)
```
这么写其实我觉得挺好，简单易懂。但是`redux`觉得，这么写一个地方还好，要是十个，百个地方都这么写，就不太好了。所以简化一下写法：
```JS
bindActionCreator(ActionCreator,dispatch);
```
然而并没有觉有多牛逼，对刚开始的人还增加阅读的难度。不过人家确实将两行代码变成了一行。


# 总结
淅淅沥沥如雨，昏昏沉沉如梦。我们完整了最基本的梦醒，对`redux`有了最基本的打开天窗说亮化的了解。如果想要深入浅出的了解他，你应该进入他的体内。别想污了，我说的是源码。
