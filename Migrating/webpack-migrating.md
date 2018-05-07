# Webpack 迁移到V4版本指南

## 需要升级的npm库列表

* "webpack": "^4.6.0",
* "webpack-cli": "^2.0.15",
* "webpack-dev-middleware": "^3.1.2",
* "webpack-hot-middleware": "^2.22.1",
* "file-loader": "^1.1.11",
* "url-loader": "^1.0.1",
* "html-webpack-plugin": "^3.2.0",
* "less": "^2.6.0",
* "less-loader": "^2.2.2",


## webpack.config.js 调整

### module 配置调整

module.loaders 变成 module.rules

```JS
module: {
    loaders: [
      {
        test: /\.css$/,
        loader: 'style!css',
      },
    ]
}

// change
// 注意新版本的loader不能省略，即：style-loader 不能写成 style
module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {loader: "style-loader"]},
          {loader: "css-loader"]},
        ]
      },
    ]
}
```


### plugins 配置调整
* remove extract-text-webpack-plugin
* remove NoErrorsPlugin
* remove UglifyJsPlugin
* remove DedupePlugin
* remove NamedModulesPlugin
* remove HashedModulesPlugin
* rename webpack.optimize.OccurenceOrderPlugin to webpack.optimize.OccurrenceOrderPlugin
* remove CommonsChunkPlugin 改用 optimization.runtimeChuck 和 optimization.splitChunks

  ```JS
  onfig.optimization = {
    minimize: true,
    // runtimeChunk: 'single', // 测试发现这个设置成true或者single首页空白，无任何报错，因此暂时不做设置。
    splitChunks: {
      chunks: 'all',
      minSize: 30000,
      minChunks: 1,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      automaticNameDelimiter: '~',
      name: true,
      cacheGroups: {
        vendors: {
          name: 'vendors',
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
      },
    },
  };
  ```




### 添加 mode
* 开发环境添加

  ```JS
  module.exports = {
    mode: 'development'
  }
  ```

* 生产环境

  ```JS
  module.exports = {
    mode: 'production'
  }
  ```

### Code Spliting
```
```

# Q&A

* `Error: Cannot find module 'webpack/lib/ConcatSource'`

  解决办法，移除：extract-text-webpack-plugin插件。（这里需要考虑如何抽离CSS）

* `OccurenceOrderPlugin has been renamed to OccurrenceOrderPlugin`

  解决办法，修改 OccurenceOrderPlugin 成 OccurrenceOrderPlugin

* `ERROR in ./assets/images/icons/search.svg`

  详细报错信息如下：

  ```
  ERROR in ./assets/images/icons/search.svg
  Module build failed: TypeError: Cannot read property 'context' of undefined
      at Object.loader (/Users/drew/Sites/my-site/node_modules/file-loader/dist/index.js:34:49)
  @ ./styles.sass 6:40863-40906
  ```

  解决办法：升级 `url-loader` 和 `file-loader`

  