# html 规范

## 文档规范

非特殊需求情况下，文档声明必须是 html5 的文档类型：

```html
<!DOCTYPE html>
```

## 脚本加载规范

非特殊需求情况下，脚本的加载应该放在:`<body/>`后面。

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>

</body>
<!-- js files -->
<script src="./jquery.js"></script>
<script src="./xxx.js"></script>
</html>
```

- 常规：顺序加载（推荐）

如果存在多个 js 文件，并且 js 文件之间有相互依赖关系，则需要依次存放 js 文件，被依赖文件在依赖文件的前面。

**注意：控制 js 文件数量**

- 延迟加载：defer

> 增加`defer`表示脚本会被延迟到整个页面都解析完后再运行，但是需要注意的是下载依旧会按从上到下的顺序。
>
> 另外如果存在多个延迟脚本，理论上是按从上到下的顺序先后执行，但是在现实当中，延迟脚本并不一定会按照顺序执行。所以不建议有多个`defer`脚本.
>
> `defer`只支持外部引入脚本。H5 明确规定，如果内嵌脚本使用`defer`将直接被忽略

非特殊需求情况下，不建议设置多个带`defer`标签的`<script></script>`脚本

```html
<script type="text/javascript" src="./example.js" defer></script>
```

- 异步加载: async

> 和 `defer`一样都是先下载后执行，和`defer`最大的区别是执行不会从上到下。标记为`async`的脚本不保证按照指定它们的先后顺序执行。

非特殊需求情况下，不推荐使用，如果处理不好会增加生产环境中引用未加载完成脚本的接口产生不可完全重现的错误，而且排查难度巨大。

```html
<script type="text/javascript" src="./example.js" async></script>
```

## 代码风格

代码风格通过`prettier`格式化插件自动格式化。

- 缩进与换行

使用 2 个空格做为一个缩进层级。

```html
<ul>
  <li>first</li>
  <li>second</li>
</ul>
```

- 每行不得超过 `120` 个字符。

过长的代码不容易阅读与维护。因此如果因为属性过多导致一个标签太长需要折行显示

```html
<!-- good -->
<div
  data-1="1"
  data-2="1"
  data-3="1"
  data-4="1"
  data-5="1"
  data-6="1"
  data-7="1"
  data-8="1"
  data-9="1"
></div>
```

## 语义化

`HTML` 标签的使用应该遵循标签的语义。

下面是常见标签语义

- p - 段落
- h1,h2,h3,h4,h5,h6 - 层级标题
- strong,em - 强调
- ins - 插入
- del - 删除
- abbr - 缩写
- code - 代码标识
- cite - 引述来源作品的标题
- q - 引用
- blockquote - 一段或长篇引用
- ul - 无序列表
- ol - 有序列表
- dl,dt,dd - 定义列表

## 标签

- 标签名必须使用小写字母。

```html
<!-- good -->
<p>Hello StyleGuide!</p>

<!-- bad -->
<p>Hello StyleGuide!</p>
```

- 标签必须成对出现或闭合

```html
<!-- good -->
<input />
<button>submit</button>
```

## 属性

- 属性顺序

属性应该按照特定的顺序出现以保证易读性；

优先顺序从上到下：

```
class
id
name
data-*
src, for, type, href, value , max-length, max, min, pattern
placeholder, title, alt
aria-*, role
required, readonly, disabled
```

class 是为高可复用组件设计的，所以应处在第一位；

id 更加具体且应该尽量少使用，所以将它放在第二位。

```html
<a class="..." id="..." data-modal="toggle" href="#">Example link</a>

<input class="form-control" type="text" />

<img src="..." alt="..." />
```

- 属性值双引号

属性值采用 `双引号` 包裹

```html
<img src="images/company_logo.png" alt="Company" />
```

- 元素 id 必须保证页面唯一

id、class 命名，在避免冲突并描述清楚的前提下尽可能短。

```html
<!-- good -->
<div id="nav"></div>
<!-- bad -->
<div id="navigation"></div>

<!-- good -->
<p class="comment"></p>
<!-- bad -->
<p class="com"></p>

<!-- good -->
<span class="author"></span>
<!-- bad -->
<span class="red"></span>
```

- 同一页面，应避免使用相同的 name 与 id。

```html
<input name="foo" />
<div id="foo"></div>
<script>
  // IE6 将显示 INPUT
  alert(document.getElementById("foo").tagName);
</script>
```

- boolean 属性

boolean 属性的存在表示取值为 true，不存在则表示取值为 false。

```html
<input type="text" disabled />

<input type="checkbox" value="1" checked />

<select>
  <option value="1" selected>1</option>
</select>
```
