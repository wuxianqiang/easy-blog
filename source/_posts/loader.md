---
title: wepack中loader的分类
date: 2019-08-17 18:50:50
tags: Webpack
---
{% asset_img banner.png 图片 %}

<!-- more -->

## loader种类
loader分为四类

分别是：
1. 前置 `pre`
2. 行内 `inline`
3. 普通 `normal`
4. 后置 `post`

## Rule.enforce
`enforce` 属性会影响 `loader` 种类。不论是普通的，前置的，后置的 loader。

可能的值有：`"pre" | "post"`
```js
module: {
    rules: [
      {
        test: /\.less$/,
        use: 'less-loader',
        enforce: 'pre'
      },
     {
        test: /\.less$/,
        use: 'css-loader'
      },
     {
        test: /\.less$/,
        use: 'style-loader',
        enforce: 'post'
      }
    ]
  },
```

指定 loader 种类。**没有值表示是普通 loader**。

## 行内loader
还有一个额外的种类"**行内 loader**"，**loader 被应用在 import/require 行内**。

**所有 loader 通过 前置, 行内, 普通, 后置 排序，并按此顺序使用。**

所有**普通 loader 可以通过在请求中加上 ! 前缀**来**忽略**（覆盖）。
```js
require('!inline-loader!./a.js')
```

所有**普通和前置 loader 可以通过在请求中加上 -! 前缀**来**忽略**（覆盖）。
```js
require('-!inline-loader!./a.js')
```

所有**普通，后置和前置 loader 可以通过在请求中加上 !! 前缀**来**忽略**（覆盖）。
```js
require('!!inline-loader!./a.js')
```

它们可在由 loader 生成的**代码中使用**。

**语法介绍**

可以在 import 语句或任何 `等同于 "import" 的方法` 中指定 loader。使用 `!` 将资源中的 loader 分开。每个部分都会相对于当前目录解析。

```js
import Styles from 'style-loader!css-loader?modules!./styles.css';
```
使用 `!` 为整个规则添加前缀，可以覆盖配置中的所有 loader 定义。(包括上面介绍的几种前缀用法)
```js
import Styles from '!style-loader!css-loader?modules!./styles.css';
```

选项可以**传递查询参数**，例如 `?key=value&foo=bar`，或者一个 JSON 对象，例如 `?{"key":"value","foo":"bar"}`。

## loader的执行顺序
所有 loader 通过 前置, 行内, 普通, 后置 排序，并按此顺序使用。
