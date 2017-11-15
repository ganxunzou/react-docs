# Redux
redux 是一个状态`state`管理器，`state`和视图是一一对应的。即通过视图我能知道对应的`state`，通过`state`也能知道对应的视图。

在react中控制视图刷新的有两个属性：`state`和`props`。`redux`在`react`中实现`state`调整对应视图刷新，是用到`react`中`props`。这里还是需要简单的区别一下。

`redux`采用发布订阅的方式将`state`和`view`连接到一起。`redux`提供了`subscribe`函数进行订阅`state`的调整。一般我们会在根视图做这个监听。
```
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
- bindActionCreators
- applyMiddleware

基础应用最常用的两个API：`createStore` 和 `combineReducers`。
