---
title: React中effect使用
date: 2019-08-25 09:51:32
tags: React
---
{% asset_img banner.png 图片 %}

之前我们已经掌握了`useState` 的使用，在 class 中，我们通过在构造函数中设置 `this.state` 为 `{ count: 0 }` 来初始化 `count` state 为 `0`：

<!-- more -->

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
```

在函数组件中，我们没有 `this`，所以我们不能分配或读取 `this.state`。我们直接在组件中调用 `useState` Hook：


```js
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 “count” 的 state 变量
  const [count, setCount] = useState(0);
```

既然我们知道了 `useState` 的作用，那么掌握 `useEffect` 就更容易，函数组件中没有生命周期，那么可以使用 `useEffect` 来替代。如果你熟悉 React class 的生命周期函数，你可以把 `useEffect` Hook 看做 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合。


```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }
  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

使用 Hook 的示例
```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

默认情况下，它在第一次渲染之后和每次更新之后都会执行。你可能会更容易接受 effect 发生在“渲染之后”这种概念，不用再去考虑“挂载”还是“更新”。React 保证了每次运行 effect 的同时，DOM 都已经更新完毕。



数据获取，设置订阅以及手动更改 React 组件中的 DOM 都属于副作用。有些副作用可能需要清除，所以需要返回一个函数，比如挂载时设置定时器，卸载时取消定时器。


```js
class Example extends Component {
  constructor (props) {
    super(props);
    this.state = {
      count: 0
    }
  }
  componentDidMount() {
    this.id = setInterval(() => {
      this.setState({count: this.state.count + 1})
    }, 1000);
  }
  componentWillUnmount() {
    clearInterval(this.id)
  }
  render() { 
    return <h1>{this.state.count}</h1>;
  }
}
```

使用 Hook 的示例
```js
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>
}
```

你可以通知 React 跳过对 effect 的调用，只要传递数组作为 `useEffect` 的第二个可选参数即可，如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（`[]`）作为第二个参数。这就告诉 React 你的 effect 不依赖于 `props` 或 `state` 中的任何值，所以它永远都不需要重复执行。



通过跳过 Effect 进行性能优化，在某些情况下，每次渲染后都执行清理或者执行 effect 可能会导致性能问题。在 class 组件中，我们可以通过在 `componentDidUpdate` 中添加对 `prevProps` 或 `prevState` 的比较逻辑解决：


```js
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

这是很常见的需求，所以它被内置到了 useEffect 的 Hook API 中。如果某些特定值在两次重渲染之间没有发生变化，你可以通知 React 跳过对 effect 的调用，只要传递数组作为 useEffect 的第二个可选参数即可：


```js
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新
```

你已经学习了 State Hook 和 Effect Hook，将它们结合起来你可以做很多事情了。它们涵盖了大多数使用 class 的用例。
