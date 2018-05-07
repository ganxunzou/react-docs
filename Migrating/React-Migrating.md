# 升级 react , react-dom
网上有一些的讲react15升级到react16升级的教程。但是很少从整体讲，都是从单个知识点讲。很多都是只讲react本身的升级，如：PropTypes 引用从react模块改成单独的prop-types模块。但是一般项目的架构都是：react+react-router+redux+react-redux+react-router-redux。所以自己整理一下完整的升级过程。

react15升级到react16涉及以下几个方面：
* react-hot-loader
* proptypes
* react-redux
* react-router

## react-hot-loader 升级
* react-hot-loader 升级最新
* webpack.base.conf.js

```JS
if (env === 'development') {
    // _loaders.unshift('react-hot');
}
```

* `.babelrc` 增加 `react-hot-loader` 配置

```JS
"plugins": [
    "react-hot-loader/babel"
]
```

## proptypes 调整

`React-PropTypes-to-prop-types.js`

1. Go to the /web directory on your project
2. `npm install global add jscodeshift`, this will add jscodeshift
3. git clone https://github.com/reactjs/react-codemod.git, this will add react-codemod
4. cd `react-codemod && npm install cd ..`
5. `jscodeshift -t <codemod-script> <path>` to transform your deprecated code. For example, if you want to fix prop-types errors in jsx files in the components directory, you can run something like: `jscodeshift -t react-codemod/transforms/React-PropTypes-to-prop-types.js app/components/**/*`

## react-redux 升级

 `react-redux@5.0.6` 版本以上

## react-router 升级
  `react-router@3.2.0` 版本以上（不超过4.0.0）

## history 升级
  `history@3.0.0` 版本以上（不超过4.0.0）

