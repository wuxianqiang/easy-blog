---
title: React中PureComponent原理
date: 2019-06-09 08:45:22
tags: React
---
PureComponent  会对 `props` 和 `state` 进行浅层比较，并减少了跳过必要更新的可能性。


`React.PureComponent` 与 `React.Component` 很相似。两者的区别在于 `React.Component` 并未实现 `shouldComponentUpdate()`，而 `React.PureComponent `中以浅层对比 `prop` 和 `state` 的方式来实现了该函数。

如果赋予 React 组件相同的 `props` 和 `state`，`render()` 函数会渲染相同的内容，那么在某些情况下使用 `React.PureComponent` 可提高性能。
```js
import React from 'react';
import ReactDOM from 'react-dom';

class Counter extends React.Component {
  state = {
    count: 0
  }
  handleClick = () => {
    this.setState({ count: this.state.count + 1 })
  }
  render() {
    return (
      <div>
        <Title title='计数器'></Title>
        <span>{this.state.count}</span>
        <button onClick={this.handleClick}>+1</button>
      </div>
    );
  }
}

class Title extends React.Component {
  render() {
    // 检查是否多次打印
    console.log('Title render')
    return (
      <p>{this.props.title}</p>
    );
  }
}

ReactDOM.render(<Counter />, document.getElementById('root'))
```
上面演示的代码中 `Counter` 组件中使用了 `Title` 组件，`Counter` 组件每次修改状态都会导致 `Title` 组件重新渲染。可以看到 `console` 语句多次打印出 `Title render`。如果不想让 `Title` 组件多次渲染，就需要让 `Title` 组件继承 `PureComponent` 组件。
# 开始使用
1. 类组件使用 `PureComponent`
```js
import React from 'react';
import ReactDOM from 'react-dom';

class Counter extends React.Component {
  state = {
    count: 0
  }
  handleClick = () => {
    this.setState({ count: this.state.count + 1 })
  }
  render() {
    return (
      <div>
        <Title title='计数器'></Title>
        <span>{this.state.count}</span>
        <button onClick={this.handleClick}>+1</button>
      </div>
    );
  }
}

class Title extends React.PureComponent {
  render() {
    console.log('Title render')
    return (
      <p>{this.props.title}</p>
    );
  }
}

ReactDOM.render(<Counter />, document.getElementById('root'))
```
2. 函数组件使用 `memo`

```js
class Counter extends React.Component {
  state = {
    count: 0
  }
  handleClick = () => {
    this.setState({ count: this.state.count + 1 })
  }
  render() {
    return (
      <div>
        <Title title='计数器'></Title>
        <span>{this.state.count}</span>
        <button onClick={this.handleClick}>+1</button>
      </div>
    );
  }
}
const Title = React.memo(props => {
  console.log('Title render')
  return <p>{props.title}</p>
})
```

3. `memo` 的原理

```js
function memo (Func) {
  class Proxy extends React.PureComponent {
    render() {
      return (
        <Func {...this.props} />
      );
    }
  }
  return Proxy
}
```

4. `PureComponent` 的原理
```js

import React, { Component } from 'react';

function shallowEqual(obj1, obj2) {
  if (obj1 === obj2) {
    return true
  }
  if (typeof obj1 !== 'object' || obj1 === null || typeof obj2 !== 'object' || obj2 === null) {
    return false
  }
  let keys1 = Object.keys(obj1)
  let keys2 = Object.keys(obj2)
  if (keys1.length !== keys2.length) {
    return false
  }
  for (const key of keys1) {
    if (!obj2.hasOwnProperty(key) || obj1[key] !== obj2[key]) {
      return false
    }
  }
  return true
}

class PureComponent extends Component {
  shouldComponentUpdate(nextProps, nextState) {
    return !shallowEqual(this.props, nextProps) || !shallowEqual(this.state, nextState)
  }
  render() {
    return this.props.children;
  }
}

export default PureComponent;
```
