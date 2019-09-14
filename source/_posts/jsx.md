---
title: 关于JSX你应该知道这些
date: 2019-09-14 17:34:59
tags: React
---

{% asset_img banner.png 图片 %}

<!-- more -->

实际上，JSX 仅仅只是 `React.createElement` 函数的语法糖。
```js
<MyButton color="blue" shadowSize={2}>
  点击
</MyButton>
```

会编译为：
```js
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  '点击'
)
```

如果没有子节点，你还可以使用自闭合的标签形式，如：
```js
// 在HTML中的写法
<div calss="sidebar"></div>

// 在React中的写法
<div className="sidebar" />
```
指定 React 元素类型
JSX 标签的第一部分指定了 React 元素的类型。
```js
React.createElement(
  'div', // 类型为div的HTML元素
  {className: 'sidebar'},
  null
)
React.createElement(
  MyButton, // 类型为MyButton的React组件
  {color: 'blue', shadowSize: 2},
  '点击'
)
```

大写字母开头的 JSX 标签意味着它们是 React 组件。这些标签会被编译为对命名变量的直接引用，所以，当你使用 JSX表达式时，`Foo` 必须包含在作用域内。
```js
const MyComponents = {
  Foo : function Foo (props) {
    return <div>Foo</div>;
  }
}

// Foo 不在当前作用域内。
<Foo />
function Foo () {
  return <div>Foo</div>;
}

// Foo 在当前作用域内。
<Foo />
```
## 在 JSX 类型中使用点语法

在 JSX 中，你也可以使用点语法来引用一个 React 组件。当你在一个模块中导出许多 React 组件时，这会非常方便。
```js
const MyComponents = {
  Foo : function Foo (props) {
    return <div>Foo</div>;
  },
  Bar : function Bar (props) {
    return <div>Bar</div>;
  }
}

<MyComponents.Foo />
<MyComponents.Bar />
```

用户定义的组件必须以大写字母开头
```js
const obj = {
  Foo : function Foo (props) {
    return <div>Foo</div>;
  },
  Bar : function Bar (props) {
    return <div>Bar</div>;
  }
}

// 错误写法
<obj.Foo />
// 正确写法
let MyComponent = obj.Foo
<MyComponent />
```
## Props 默认值为 “True”

如果你没给 `prop` 赋值，它的默认值是 `true`。以下两个 JSX 表达式是等价的：
```js
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

## 属性展开

如果你已经有了一个 `props` 对象，你可以使用展开运算符 `...` 来在 JSX 中传递整个 `props` 对象。以下两个组件是等价的：
```js
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```
## JSX 中的子元素

布尔类型、Null 以及 Undefined 将会忽略。

`false`, `null`, `undefined`, and `true` 是合法的子元素。但它们并不会被渲染。以下的 JSX 表达式渲染结果相同：
```js
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```
如数字 `0`，仍然会被 React 渲染。
