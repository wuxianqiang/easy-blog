---
title: 彻底理解AMD和CMD
date: 2019-07-21 22:10:39
tags: Node
---

{% asset_img banner.png 图片 %}

一个模块就是实现特定功能的文件，有了模块，我们就可以更方便地使用别人的代码，想要什么功能，就加载什么模块。模块开发需要遵循一定的规范，各行其是就都乱套了。

<!-- more -->

## AMD

AMD 规范是 CommonJS 模块规范的一个延伸，它的模块定义如下：

```js
define(id?, dependencies?, factory);
```

它的模块 `id` 和依赖是可选的，与Node模块相似的地方在于 `factory` 的内容就是实际代码的内容。

下面是在 Node 是使用 AMD 规范的例子，该例子使用了第三方库  `requirejs`。

```js
// a.js
define(function() {
  return 'hello world'
});
```

```js
// b.js
define(['a'], function(data) {
  let exports = {}
  exports.sayHello = function () {
    console.log(data)
  }
  return exports
});
```

```js
// index.js
const requirejs = require('requirejs');
requirejs.config({
  nodeRequire: require
});
requirejs(['b'], function (obj) {
  obj.sayHello()
});
```

于 commonJS 不同之处在于，AMD 模块需要用 `define` 来明确定义一个模块，而在 Node 实现中是隐式包装的，它们的
目的是进行作用域隔离，仅在需要的时候被引入，避免掉过去那种通过全局变量或者全局命名空间的方式，以免变量污染和不小心被修改。另一个区别则是内容需要通过返回的方式实现导出。

## CMD

CMD规范由国内的玉伯提出，与AMD规范的主要区别在于定义模块和依赖引入的部分。AMD需要在声明模块的时候指定所有的依赖(依赖前置)，通过形参传递依赖到模块内容中：

```js
define(['dep1', 'dep2'], function (dep1, dep2) {
	return function () {};
});
```

与AMD模块规范相比，CMD模块更接近于Node对CommonJS规范的定，在依赖部分，CMD支持动态引入(就近依赖)，示例如下。

下面是在 Node 是使用 CMD 规范的例子，该例子使用了第三方库  `seajs`。

```js
// a.js
define(function() {
  return 'hello world'
});
```
```js
// b.js
define(function(require) {
  let data = require('./a')
  let exports = {}
  exports.sayHello = function () {
    console.log(data)
  }
  return exports
});
```
```js
// index.js
require('seajs');
let obj = require('./b')
obj.sayHello()
```

AMD和CMD最大的区别是对依赖模块的执行时机处理不同。

```js
define(function(require, exports, module) {
// The module code goes here
});
```

`require` 、 `exports` 和 `module` 通过形参传递给模块，在需要依赖模块时，随时调用 require() 引入即可。三个参数表示的意思和commonJS规范相同。
