---
title: 使用React.Suspense显示loading效果
date: 2019-08-17 18:49:34
tags: React
---
{% asset_img banner.png 图片 %}

<!-- more -->

1. `<React.Suspense>`需要和`React.lazy()`配合使用
2. `React.lazy()`需要和`import()`动态引入语法配合使用
3. 不能在服务端渲染使用


在组件没有加载出来的时候显示loading效果，加载完成之后正常显示。
```js
import React from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent.js'));

function MyComponent (params) {
  return (
    <div>
      <React.Suspense fallback={<div>Loading</div>}>
        <OtherComponent />
      </React.Suspense>
    </div>
  )
}
export default MyComponent
```
`React.lazy` 接受一个函数，这个函数需要动态调用 `import()`。它必须返回一个 `Promise`，该 Promise 需要 resolve 一个 `defalut` export 的 React 组件。

```js
import React, { Component } from 'react';

class OtherComponent extends Component {
  render() { 
    return (
      <div>
        hello world
      </div>
    );
  }
}

export default OtherComponent; // 必须是默认导出
```
`fallback` 属性接受任何在组件加载过程中你想展示的 React 元素。你可以将 `Suspense` 组件置于懒加载组件之上的任何位置。你甚至可以用一个 `Suspense` 组件包裹多个懒加载组件。
