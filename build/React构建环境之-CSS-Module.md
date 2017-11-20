# CSS 在构建环境中需要思考的几个点
- 样式污染问题
- 样式中图片引用问题
- 样式文件抽离    

## 样式污染问题
`html` 中的样式没有块作用域的概念，所以一旦设置了样式就是全局生效。那怎么解决样式污染呢，唯一的方式就是样式名称做到唯一性。现在最流行的方式就是`css modules`，也有另外一种方式`CSS-IN-JS`。因为嵌入`CSS`可能导致`JS`文件变大，丢失了`CSS`缓存策略，因此个人不太喜欢。（PS:`css` 样式污染问题还有一种方式就是规定开发规范，比如：`路径名称-样式名称`）

## 样式中图片引用
样式中引用图片，一般的做法是设置个`limit`，当图片的大小小于`limit`就直接内联到css中，当图片的大小超过`limit`就单独成文件。现在还有一种比较有意思的做法，就是把图片打包成`sprite`图。有`css`经验的人都知道，`sprite`（雪碧）技术能有效的减小请求次数。原理就是做一张大图，然后用`css`去裁剪。

## 样式抽离
在生产环境，一般希望把所有的`css`打包到一个`style.css`文件。（PS： 多个entry或者Code Splitting的情况另说），`css`和`JS`分离的好处个人认为主要是两个：有独立缓存 和 js文件体积。

# Webpack CSS Module 配置

```JS
use: [
    'style-loader',
    {
        loader: 'css-loader',
        options: {
            modules: true,//启用CSS module
            sourceMap: true,
            localIdentName: '[name]__[local]--[hash:base64:5]',
        },
    },
]
```
启用`module`之后，React中的写法必须要写成强制引用的方式。即
```CSS
import style from './style.css';
//...
<div className={style.main}></div>
// or
var classname = classnames({style.main,style.main1})
<div className={classname}></div>
```

如果直接写成下面方式，`css-module` 默认他是全局 `css style` ，不会做任何处理（在`style.css`中的样式也不会加载进去）
```CSS
import './style.css';
//...
<div className="main"></div>
```

个人不太喜欢这种写法`<div className={style.main}></div>`，因此我使用了一个`babel`插件。

在`.babelrc`增加如下插件：（特别注意：`generateScopedName` 和 `webpack` 中的 `css-loader`的 `localIdentName` 要对应上）
```JS
["react-css-modules",
    {
        "webpackHotModuleReloading":true,
        "generateScopedName":"[name]__[local]--[hash:base64:5]"
    }
],
```
使用 `babel-plugin-react-css-modules`也有一个小问题。它提供了一个`styleName`标签。也就是说他有两个定义样式的标签：`className`和`styleName`。
```
<div className="global-css" styleName="local-css"></div>
```
这个小问题就是要调整写法，这个对未来的升级可能会有一定的影响。PS：要是`<div className="local-css" styleName="global-css"></div>`这种方式该多好。

# Webpack Extract CSS 配置
webpack提供了一个样式抽离插件：`extract-text-webpack-plugin`。这个插件使用起来比较简单，配置一下就可以了。
```JS
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const extractCSS = new ExtractTextPlugin('[name].css');

module: {
 rules: [
    {
    test: /\.css$/i,
    use: extractCSS.extract({
        fallback: 'style-loader',
        use: [{
            loader: 'css-loader',
            options: {
                modules: true,
                sourceMap: true,
                minimize: true,
                localIdentName: '[name]__[local]--[hash:base64:5]',
            },
        }],
        //	publicPath: '/build'
    }),
    },
 ]
}
```
这边曾遇到一个问题，我想把`css`文件放到`css`目录里面，结构发现里面对外`img`的引用就找不到的。默认抽离的`css`文件中对图片的引用是相对目录的。抽离编译后的`css`中对图片的引用是`./img/aaa.png`这种方式（PS：这里的`img`因为我配置的`url-loader`输出到`img`目录，详见下面章节：`css 中图片抽离`）。


# Webpack CSS 图片抽离
经常会在CSS中引用图片（PS：如果是小图标，建议用字体图标，类似的图标图现在很多，国内最好的应该是阿里巴巴提供的：[http://www.iconfont.cn/](http://www.iconfont.cn/)）。图片一般较大，如果直接打包到CSS中，很影响首次加载的时间。另外图片自身的缓存策略也失效了。所以一般情况我们把图片从css中抽离出来，抽离的条件就是设置一个`limit`大小边界。下面是webpack中图片从css中抽离的配置。
```JS
{
test: /\.(eot|woff|woff2|svg|ttf|png|jpg|jpeg)$/,
use: [{
    loader: 'url-loader',
    options: {
        limit: 10000, // 10K
        fallback: 'file-loader', // default
        name: '[name]-[hash:8].[ext]',
        // publicPath: 'assets/',
        outputPath: './imgs/',
        useRelativePath: false, // true : outputPath 失效
    },
}],
},
```
这里简单的说一下`url-loader`和`file-loader`。`url-loader` 做第一步拦截，然后设置一些拦截条件,如果资源超过`limit`限制。将操作权限转移到`file-loader`，并把`options`以`queryString`的方式传递给`file-loader`。当然如果你想自己指定处理的`loader`也是可以的，配置`fallback：custom-loader`即可。

## CSS 图片抽离之Sprite图
没实际的验证过，等有空在验证。用到的是`postcss`平台中的`postcss-sprites` 插件