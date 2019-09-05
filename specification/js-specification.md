# javascript 开发规范

## 文档规范

- 编码

JavaScript 文件使用无 BOM 的 UTF-8 编码。

- 常量

对于

## 代码风格

- 缩进

- 空格

```js
// good
if (condition) {
}

while (condition) {}

function funcName() {}

// bad
if (condition) {
}

while (condition) {}

function funcName() {}
```

- 换行

每行不得超过 120 个字符。

```js
// good
if (
  (user.isAuthenticated() &&
    user.isInRole("admin") &&
    user.hasAuthority("add-admin")) ||
  user.hasAuthority("delete-admin")
) {
  // Code
}
```

## 命名

- 声明

  优先使用 `let`，`const` 声明变量和常量。不推荐使用`var`

- 变量

采用首字母小写的驼峰命名法。

```js
var loadingModules = {};
```

- 常量

使用 全部字母大写，单词间下划线分隔 的命名方式。

```js
var HTML_ENTITY = {};
```

- 函数

采用首字母小写的驼峰命名法。

```js
function stringFormat(source) {}
```

- 函数参数

与变量命名相同，采用首字母小写的驼峰命名法。

```js
function hear(theBells) {}
```

- 类

采用首字母大写的驼峰命名法。类名命名采用名词。

```js
// class es6
class Person {}

// function
function Person() {}
```

- 类方法

类的`方法 / 属性`使用 CamelCase 命名法

```js
// class es6
class Person {
  eatFood = () => {};
}

// function
function Person() {}
Person.prototype.eatFood = function() {};
```

- 枚举变量

使用 Pascal 命名法，枚举的属性 使用 全部字母大写，单词间下划线分隔 的命名方式。

```js
var TargetState = {
  READING: 1,
  READED: 2,
  APPLIED: 3,
  READY: 4,
};
```

- bool 类型

简单是否采用：`is`前缀；是否含有采用：`has`前缀；检测是否存在采用:`check`前缀；

---

## 注释

- 单行注释

必须独占一行。// 后跟一个空格，缩进与下一行被注释说明的代码一致。

- 多行注释

避免使用 `/*...*/` 这样的多行注释。有多行注释内容时，使用多个单行注释。

- 文档化注释（jsdoc）

JSDoc 注释一般应该放置在方法或函数声明之前，它必须以`/**`开始，以便由 JSDoc 解析器识别。其他任何以`/*`，`/***`或者超过 2 个星号的注释，都将被 JSDoc 解析器忽略。

注释规范示例：

```js
/**
* @author llying
* @constructor Person
* @description 一个Person类
* @extends 标记类的继承信息。
* @see The <a href=”#”>llying</a >.
* @example new Parent(“张三”,15);
* @since version 0.1
* @param {String} username 姓名
* @param {number} age 年龄
*/
function Person(username,age)
{
    /**
    * @description {Sting} 姓名
    * @field
    */
    this.username = username;

    /**
    * @description {number} 年龄
    * @field
    */
    this.age = age;

    /**
    * @description 弹出say内容
    * @param {String} content 内容
    */
    this.say = function(content)
    {
        alert(this.username+” say :”+content);
    }

    /**
    * @description 返回json格式的对象
    * @return {String} json格式
    * @see Person#say
    */
    this.getJson = function(){
        return “{name:”+this.username+”,age”+this.age+”}”;
    }
}
```

常见的注释：

```
@param 指定参数名和说明来描述一个函数参数

@returns 描述函数的返回值

@author 指示代码的作者

@deprecated 指示一个函数已经废弃，而且在将来的代码版本中将彻底删除。要避免使用这段代码

@see 创建一个HTML链接，指向指定类的描述

@version 指定发布版本

@requires 创建一个HTML链接，指向这个类所需的指定类

@throws @exception 描述函数可能抛出的异常的类型

{@link} 创建一个HTML链接，指向指定的类。这与@see很类似，但{@link}能嵌在注释文本中

@fileoverview 这是一个特殊的标记。如果在文件的第一个文档块中使用这个标记，则指定该文档块的余下部分将用来提供这个文件的概述

@class 提供类的有关信息，用在构造函数的文档中

@constructor 明确一个函数是某个类的构造函数

@type 指定函数的返回类型

@extends 指示一个类派生了另一个类。JSDoc通常自己就可以检测出这种信息，不过，在某些情况下则必须使用这个标记

@private 指示一个类或函数是私有的。私有类和函数不会出现在HTML文档中，除非运行JSDoc时提供了–private命令行选项

@final 指示一个值是常量值。要记住JavaScript无法真正保证一个值是常量

@ignore JSDoc忽略有这个标记的函数
```

类型定义：

对于基本类型 {string}, {number}, {boolean}，首字母必须小写。

| 类型定义          | 语法示例                           | 解释                                      |
| ----------------- | ---------------------------------- | ----------------------------------------- |
| String            | {string}                           | --                                        |
| Number            | {number}                           | --                                        |
| Boolean           | {boolean}                          | --                                        |
| Object            | {Object}                           | --                                        |
| Function          | {Function}                         | --                                        |
| RegExp            | {RegExp}                           | --                                        |
| Array             | {Array}                            | --                                        |
| Date              | {Date}                             | --                                        |
| 单一类型集合      | {Array.<string>}                   | string 类型的数组                         |
| 多类型            | {(number ｜ boolean)}              | 可能是 number 类型, 也可能是 boolean 类型 |
| 允许为 null       | {?number}                          | 可能是 number, 也可能是 null              |
| 不允许为 null     | {!Object}                          | Object 类型, 但不是 null                  |
| Function 类型     | {function(number, boolean)}        | 函数, 形参类型                            |
| Function 带返回值 | {function(number, boolean):string} | 函数, 形参, 返回值类型                    |
| 参数可选          | @param {string=} name              | 可选参数, =为类型后缀                     |
| 可变参数          | @param {...number} args            | 变长参数, ...为类型前缀                   |
| 任意类型          | {\_}                               | 任意类型                                  |
| 可选任意类型      | @param {\_=} name                  | 可选参数，类型不限                        |
| 可变任意类型      | @param {...\*} args                | 变长参数，类型不限                        |

- 文件注释

文件顶部必须包含文件注释，用 @file 标识文件说明。

```js
/**
 * @file Describe the file
 */
```
