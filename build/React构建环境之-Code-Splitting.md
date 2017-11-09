# React构建环境之-Code-Splitting
代码拆分在React应用的是相当必要的。试想如果将所有的业务应用都打包到一个JS中,不仅JS文件会变得超大，另外有个致命的问题就是每次一个业务场景的变更，都需要重新打包整个应用，这将会产生很大的投产风险。因此需要的是将业务代码分离，按需加载。


> 插播一段说明 ： 代码分离分两种：开发的代码分离 和 按需加载功能分离。我们这边说讲的就是：按需价值功能分离。其中开发时的代码分离，JS中已经有CommonJS和ES6 的Module 实现了。

代码分离一般需要考虑下三类：
 - **主应用分离**：应用程序的壳，没有具体的业务逻辑
 - **公用代码分离**：程序运行以来环境，如：React，React-Router，自定义组件库等。
 - **业务代码分离**：具体的业务页面

## 公共代码分离

webpack 提供了功能代码分离插件：`new webpack.optimize.CommonsChunkPlugin()`，我们只要简单的在`webpack.confg.js`配置即可。配置如下：
```
const config = {
    entry: {
        // 增加一个entry，配置要公共分离的库
        vendor: ['react', 'react-dom'],
        App: './src/index.jsx',
    },
    plugins: [
        // 抽离出公共模块到独立的js
        new webpack.optimize.CommonsChunkPlugin({
            name: 'vendor', // 对应 entry
            filename: '[name]-[chunkhash:8].bundle.js',
            minChunks: Infinity,
        }),
    }
}
```

## 业务代码分离
主程序代码分离和业务代码分离一般在一起处理。业务代码分离需要用到`webpack 2`提供的动态加载。（PS，一般会选择和Router一起用。不过也可以不使用router）

```
import React, { Component } from 'react';
// import PropTypes from 'prop-types';
import style from './App.css';

class App extends Component {
  dynamicImport(e, type) {
    // 代码分离
    import(`./${type}/index` /* webpackChunkName:${type} */).then((foo) => {
      console.log(foo);
      this.foo = foo;
      this.forceUpdate();
    });
  }
  render() {
    return (
            <div className={style.main}>
              <h1>dynamic add script</h1>
                <div>
                  <button onClick={e => this.dynamicImport(e, 'header')}>addHeader</button>
                  <button onClick={e => this.dynamicImport(e, 'body')}>addBody</button>
                  <button onClick={e => this.dynamicImport(e, 'footer')}>addFooter</button>
                </div>
                {this.foo ? <this.foo.default /> : null}
            </div>
    );
  }
}
App.propTypes = {
  // name: PropTypes.string,
};

export default App;
```

### Webpack2 写法提升
Webpack 1的写法
```
require.ensure([], function(require) {
  var foo = require("./module");
});
```
Webpack 2 返回的是 `Promise`
```
function onClick() {
  import("./module").then(module => {
    return module.default;
  }).catch(err => {
    console.log("Chunk loading failed");
  });
}
```

Webpack 2 的写法更加优雅一些，不过如果你需要在打包的时候指定名字的，例如：`[name]-bundle.js`，Webpack 2 的模式就很头疼了。

比如上面我动态加载三个文件：`header`,`body`,`footer`,我希望打包的是生成的名字分别是：`header.bundle.js`,`body.bundle.js`和 `footer.bundle.js`。

- webpack 1 

  只需要在`webpack.config.js`配置文件的`output`节点增加`chunkFilename`即可。
  ```
  const config = {
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist'),
        chunkFilename: '[name].bundle.js',//增加这个
    }
  }
  ```

- webpack 2 
在Webpack 2 中竟然无法实现这个功能，因为在Webpack 2 中，chunkname 是通过注释来注入的 `/* webpackChunkName:header */` ,因为是注释代码，还不能动态拼接。感觉好奇葩的设计。
    ```
    import(`./${type}/index` /* webpackChunkName:header */).then((foo) => {
        console.log(foo);
        this.foo = foo;
        this.forceUpdate();
        });
    ```

### 开发环境配置
如果你和我一样用的也是`Babel` 转码工具，那还需要在`.babelrc` 中增加一个插件。
```
    "plugins": [
        "syntax-dynamic-import"
    ]
```