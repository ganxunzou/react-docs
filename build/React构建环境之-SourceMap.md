# React 构建环境之-SourceMap
`sourcemap` 就像他的翻译：源码映射。为什么要用`sourcemap` 呢？先来看看目前我们的构建环境做了那些事情：
- 压缩代码：为了减小文件体积，我们需要压缩代码
- 混淆代码：为了源码安全，我们需要混淆代码
- ES6 -> ES5: 为了兼容浏览器，我们需要将ES6编写的程序，转成ES5的代码。

代码转换和混淆压缩后，如果代码出现问题将会很难排查。这时候`sourcemap`就体现了他的价值。`sourcemap`的作用就是在报错的时候，我们可以找到对应的代码（未压缩和混淆前的代码）

## Webpack SourceMap 配置
Webpack SourceMap 配置可以有两种方式，一种是直接在`webpack.config.js`中增加:`devtool: 'eval-source-map'` 或者在 `plugins` 节点增加 `new webpack.SourceMapDevToolPlugin()` 插件。`SourceMapDevToolPlugin` 插件的控制更加粒度化。使用`devtool` 方式注意选择正确的测试环境和生产环境的配置。

## Webpack 生产环境排除 `vendor` 源码
`vendor` 的说明详见：[React 构建环境之-Code-Splitting](./React构建环境之-Code-Splitting.md)。

生产环境我们其实不一定要看官方库的源码，例如：React，React-Router等。要实现这个功能就必须用 `SourceMapDevToolPlugin` 。详细配置如下：
```
new webpack.SourceMapDevToolPlugin({
      filename: '[file].map', // 请注意这里，不能写成[name].js.map,这种方式生成的map文件是个空文件
      exclude: ['vendor'], // 将vendor 排除出去
    }),
```

## SourceMap 自定义文件名
- `devtool` 方式

   在 `webpack.config.js`的`output`节点增加 `sourceMapFilename: '[name].js.map'`

- `SourceMapDevToolPlugin`
  ```
    new webpack.SourceMapDevToolPlugin({
      filename: '[file].map', // 请注意这里，不能写成[name].js.map,这种方式生成的map文件是个空文件（bug呀）
    }),
  ```

## 其他注意事项
`SourceMap` 还需要浏览器的支持，所以浏览器不支持也会狗带。
- Chrome 浏览器的配置：
开发者界面 -> Setting ->  Sources -> Enable JavaScript source maps 勾选上

