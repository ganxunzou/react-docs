# React-Redux
首先声明这个是`redux`官方提供的针对`react`的版本。所以呢如果你是`react`应用，那肯定得选`react-redux`。

不过在`react-redux`之前，还要需要先了解`redux`，详见我的另一篇 [redux](./redux.md) 。

`react-redux`提供了以下几个API，我们来聊聊核心的两个API:`Provider`和`connect`

- `Provider` 
- `createProvider`
- `connectAdvanced`
- `connect`


## Provider
```JS
import React from 'react';
import ReactDOM from 'react-dom';
import { AppContainer } from 'react-hot-loader';
import { Provider } from 'react-redux';
import App from './App';
import store from './store';

const render = (Component) => {
  ReactDOM.render(
		<Provider store={store}> 
			<AppContainer>
					<Component />
			</AppContainer>
		</Provider>,
    document.getElementById('app'),
  );
};

render(App);

// Webpack Hot Module Replacement API
if (module.hot) {
  module.hot.accept('./App', () => { render(App); });
}

// store.subscribe(() => {
// 	render(App);
// });

```
看过前面的我写的`redux`文章的应该知道，`disptach`需要自己订阅才能刷新视图，而且之前 `store` 属性只传递给了根视图，如果子组件需要获取store需要在我们的根视图中逐层往下传，或者顶一个`context`。

`Provider`的作用就是提供子组件获取`store`,采用的原理就是`react`的`context`。来看看`Provider`的源码就一目了然了。
```JS
export function createProvider(storeKey = 'store', subKey) {
    const subscriptionKey = subKey || `${storeKey}Subscription`

    class Provider extends Component {
        getChildContext() {
          return { [storeKey]: this[storeKey], [subscriptionKey]: null }
        }

        constructor(props, context) {
          super(props, context)
          this[storeKey] = props.store;
        }

        render() {
          return Children.only(this.props.children)
        }
    }

    if (process.env.NODE_ENV !== 'production') {
      Provider.prototype.componentWillReceiveProps = function (nextProps) {
        if (this[storeKey] !== nextProps.store) {
          warnAboutReceivingStore()
        }
      }
    }

    Provider.propTypes = {
        store: storeShape.isRequired,
        children: PropTypes.element.isRequired,
    }
    Provider.childContextTypes = {
        [storeKey]: storeShape.isRequired,
        [subscriptionKey]: subscriptionShape,
    }

    return Provider
}

export default createProvider()
```

## connect 基础
`react-redux`将组件分成两种：UI组件和容器组件。UI组件只负责展示，UI组件内部的数据和操作（例如：onClick）都由容器组件提供。这样做的好处是UI组件可以高度复用。

`react-redux`规定，所有的UI组件由开发者自己定义，容器组件由`react-redux`通过`connect`函数生成。

`connect` 函数携带两个参数：`mapStateToProps`和`mapDispatchToProps`。这两个参数都是纯函数。两个函数返回值必须是简单对象，即：`{key:value}`。

`mapStateToProps`是一个携带`state`和`ownProps`两个属性的纯函数（PS：看源码`ownProps`好像是打酱油的，完全没用到。），该函数负责`redux`的`state`到UI组件`props`的映射，简单说就是`state`改变会导致UI组件的`props`改变，`props`改变会重新刷新视图。

`mapDispatchToProps` 是一个携带`dispatch`和`ownProps`两个属性的纯函数（PS：同样`ownProps`好像是打酱油的），该函数负责的UI组件的操作（注意不是`dispatch`）到`props`的映射。UI组件的操作一般是用户行为，例如：`onClick`，`onChange`，用户行为内部用`dispatch(action)`的方式更新数据。

下面看一个简单的示例：
```JS
import React, { Component } from 'react';
import { connect } from 'react-redux';

import './App.css';
import TodoView from './components/todo';
import TodoAdd from './components/todo/add';
import { addTodo, toggleTodo, delTodo } from './actions/todo';


class App extends Component {
  render() {
		// dispatch,
		const { todolist, onAdd, onDelete, onChangeStatus } = this.props;
    return (
         <div styleName="main">
					<TodoAdd onAdd={(text) => { onAdd(text); }} />
					<TodoView todolist={todolist}
							onChangeStatus={todo => onChangeStatus(todo)}
							onDelete={todo => onDelete(todo)}
					/>
         </div>
    );
  }
}
// State => Props
const mapStateToProps = (state) => {
	return { todolist: state.todo };
};

// dispatch => Props
const mapDispatchToProps = dispatch => ({
		onAdd: (text) => {
			dispatch(addTodo(text));
		},
		onChangeStatus: (todo) => {
			dispatch(toggleTodo(todo));
		},
		onDelete: (todo) => {
			console.log('delete', todo);
			dispatch(delTodo(todo));
		},
	});

export default connect(mapStateToProps, mapDispatchToProps)(App);
```

## connect 高阶
看过我写的redux文章的，肯定会有一个疑问，就是`subscribe`哪里去了？以前我们我们在根组件上订阅`dispatch`，然后更新根视图，现在没地方看到`subscribe`，那怎么实现刷新视图的呢？

这个问题正是`connect`的第二个功能。容器组件订阅`dispatch`。大家可以去看看源码，这里就直接把关键源码提取出来。

`react-redux/src/utils/Subscription.js`
```JS
export default class Subscription {
  constructor(store, parentSub, onStateChange) {
    this.store = store
    this.parentSub = parentSub
    this.onStateChange = onStateChange
    this.unsubscribe = null
    this.listeners = nullListeners
  }

  addNestedSub(listener) {
    this.trySubscribe()
    return this.listeners.subscribe(listener)
  }

  notifyNestedSubs() {
    this.listeners.notify()
  }

  isSubscribed() {
    return Boolean(this.unsubscribe)
  }

  trySubscribe() {
    if (!this.unsubscribe) {
      this.unsubscribe = this.parentSub
        ? this.parentSub.addNestedSub(this.onStateChange)
        : this.store.subscribe(this.onStateChange) // 这是关键源码
 
      this.listeners = createListenerCollection()
    }
  }

  tryUnsubscribe() {
    if (this.unsubscribe) {
      this.unsubscribe()
      this.unsubscribe = null
      this.listeners.clear()
      this.listeners = nullListeners
    }
  }
}
```

`react-redux/src/components/connectAdvanced.js`
```JS
onStateChange() {
    this.selector.run(this.props)

    if (!this.selector.shouldComponentUpdate) {
      this.notifyNestedSubs()
    } else {
      this.componentDidUpdate = this.notifyNestedSubsOnComponentDidUpdate
      this.setState(dummyState)  // 刷新视图
    }
  }
```

# 总结
淅淅沥沥如雨，昏昏沉沉如梦。我们完整了最基本的梦醒，对`react-redux`有了最基本的打开天窗说亮化的了解。如果想要深入浅出的了解他，你应该进入他的体内。别想污了，我说的是源码。

