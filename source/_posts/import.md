---
title: 代码分割
date: 2020-02-23 10:24:25
tags: React
---
{% asset_img banner.png 图片 %}

### 打包
大多数 React 应用都会使用 Webpack，Rollup 或 Browserify 这类的构建工具来打包文件。 打包是一个将文件引入并合并到一个单独文件的过程，最终形成一个 “bundle”。 接着在页面上引入该 bundle，整个应用即可一次性加载。

<!-- more -->
演示示例

```js
/ math.js
export function add(a, b) {
  return a + b;
}
```
```js
// app.js
import { add } from './math.js';

console.log(add(16, 26)); // 42
```
打包后文件：
```js
function add(a, b) {
  return a + b;
}

console.log(add(16, 26)); // 42
```
### 代码分割

打包是个非常棒的技术，但随着你的应用增长，你的代码包也将随之增长。尤其是在整合了体积巨大的第三方库的情况下。你需要关注你代码包中所包含的代码，以避免因体积过大而导致加载时间过长。


为了避免搞出大体积的代码包，在前期就思考该问题并对代码包进行分割是个不错的选择。 代码分割是由诸如 Webpack，Rollup 和 Browserify（factor-bundle）这类打包器支持的一项技术，能够创建多个包并在运行时动态加载。


对你的应用进行代码分割能够帮助你“懒加载”当前用户所需要的内容，能够显著地提高你的应用性能。尽管并没有减少应用整体的代码体积，但你可以避免加载用户永远不需要的代码，并在初始加载的时候减少所需加载的代码量。

### import()

在你的应用中引入代码分割的最佳方式是通过动态 import() 语法。

使用之前：
```js
import { add } from './math';

console.log(add(16, 26));
```
使用之后
```js
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```
当 Webpack 解析到该语法时，会自动进行代码分割。如果你使用 Create React App，该功能已开箱即用，你可以立刻使用该特性。Next.js 也已支持该特性而无需进行配置。

### React中的代码分割

React.lazy 函数能让你像渲染常规组件一样处理动态引入（的组件）。然后应在 Suspense 组件中渲染 lazy 组件，如此使得我们可以使用在等待加载 lazy 组件时做优雅降级（如 loading 指示器等）。

基于路由的代码分割

决定在哪引入代码分割需要一些技巧。你需要确保选择的位置能够均匀地分割代码包而不会影响用户体验。

一个不错的选择是从路由开始。大多数网络用户习惯于页面之间能有个加载切换过程。你也可以选择重新渲染整个页面，这样您的用户就不必在渲染的同时再和页面上的其他元素进行交互。

这里是一个例子，展示如何在你的应用中使用 React.lazy 和 React Router 这类的第三方库，来配置基于路由的代码分割。

```js
mport { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import React, { Suspense, lazy } from 'react';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```
