---
title: ES6模块的循环加载
date: 2019-09-14 17:50:50
tags: ES6
---

{% asset_img banner.png 图片 %}

<!-- more -->

先理解JS代码执行的过程。

1.引擎：负责整个JavaScript编译及执行的过程
2.编译器：负责语法分析和代码生成
3.作用域：负责收集并维护所有声明的标识符组成一系列的查询系统，确定当前执行的代码对标识符的访问权限（根据名称查找变量的一套规则）


CommonJS 模块是运行时加载（运行时执行一次），ES6 模块是编译时输出接口（编译时执行一次）。
{% asset_img loop.png 图片 %}


看看下面一个循环引用的例子。


```js
main.js

import a from './a';
console.log(a)
a.js

console.log('执行a')
import b from './b' // 开始编译b模块，代码停留在这里
console.log(b)
console.log('导入b之后')

export default '我是a模块'
b.js

console.log('执行b')
import a from './a' // 由于a模块还在编译阶段，代码并没有执行结果
console.log(a) // 变量是undefined
console.log('导入a之后')

export default '我是b模块'
```

结果
```js
// 编译阶段
b.js:1 执行b
b.js:3 undefined
b.js:4 导入a之后

// 执行阶段
a.js:1 执行a
a.js:3 我是b模块
a.js:4 导入b之后
main.js:2 我是a模块
```

ES6根本不会关心是否发生了"循环引用"，只是生成一个指向被加载模块的引用，需要开发者自己保证，取值的时候能够取到值。



模块的循环引用本身不会造成代码报错，只是取值的时候可能无法取到值，可以通过一些逻辑判断处理即可。
