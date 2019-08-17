---
title: vue使用slot分发内容与react使用prop分发内容
date: 2019-08-17 18:41:17
tags: Vue
---
{% asset_img banner.png 图片 %}

<!-- more -->

# vue

将 `<slot>` 元素作为承载分发内容的出口
```js
// layout.vue
<div class="container">
  <main>
    <slot></slot>
  </main>
</div>
```
当组件渲染的时候，`<slot></slot>` 将会被替换该组件起始标签和结束标签之间的任何内容
```js
// hone.vue
<div class="container">
  <layout>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
 </layout>
</div>
```
# react
每个组件都可以获取到 `props.children`。它包含组件的开始标签和结束标签之间的内容。
```js
class Layout extends Component {
  render() {
    return (
      <div className="container">
        <main>
          {this.props.children}
        </main>
      </div>
    );
  }
}
```
`props` 是 React 组件的输入。它们是从父组件向下传递给子组件的数据。
```js
function Home (params) {
  return (
    <>
      <Layout>
        <p>A paragraph for the main content.</p>
        <p>And another one.</p>
      </Layout>
    </>
  )
}
```
# vue
有时我们需要多个插槽。对于这样的情况，`<slot>` 元素有一个特殊的特性：`name`。这个特性可以用来定义额外的插槽：
```js
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```
一个不带 name 的 `<slot>` 出口会带有隐含的名字“default”

在向具名插槽提供内容的时候，我们可以在一个 `<template>` 元素上使用 `v-slot` 指令，并以 `v-slot` 的参数的形式提供其名称：
```js
<layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</layout>
```
现在 `<template>` 元素中的所有内容都将会被传入相应的插槽。任何没有被包裹在带有 `v-slot` 的 `<template>` 中的内容都会被视为默认插槽的内容。

# react
注意：组件可以接受任意 props，包括基本数据类型，React 元素以及函数。
```js
class Layout extends Component {
  render() {
    return (
      <div className="container">
        <header>
          {this.props.header}
        </header>
        <main>
          {this.props.children}
        </main>
        <footer>
          {this.props.footer}
        </footer>
      </div>
    );
  }
}
```
少数情况下，你可能需要在一个组件中预留出几个“洞”。这种情况下，我们可以不使用 `children`，而是自行约定：将所需内容传入 `props`，并使用相应的 `prop`。
```js
function Home (params) {
  return (
    <>
      <Layout
        header={
          <h1>Here might be a page title</h1>
        }
        footer={
          <p>Here's some contact info</p>
        } >
        <p>A paragraph for the main content.</p>
        <p>And another one.</p>
      </Layout>
    </>
  )
}
```
React 元素本质就是对象（object），所以你可以把它们当作 props，像其他数据一样传递。这种方法可能使你想起别的库中“槽”（`slot`）的概念，但在 React 中没有“槽”这一概念的限制，你可以将任何东西作为 `props` 进行传递。
