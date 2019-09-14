---
title: 不要在 render 方法中使用 HOC
date: 2019-09-14 17:46:52
tags: React
---

{% asset_img banner.png 图片 %}

<!-- more -->

React 的 diff 算法（称为协调）使用组件标识来确定它是应该更新现有子树还是将其丢弃并挂载新子树。如果从 render 返回的组件与前一个渲染中的组件相同（===），则 React 通过将子树与新子树进行区分来递归更新子树。如果它们不相等，则完全卸载前一个子树。



例子：下面有两个计数器，第一个是正常组件，第二个是函数包裹的高阶组件，点第一个的时候第二个组件的状态会清空。


```js
function WidthWrapper (WrappedComponent) {
  return class extends React.Component {
    componentDidMount() {
      console.log('页面加载完成')
    }
    render() {
      return (
        <WrappedComponent />
      );
    }
  }
}

class Example extends React.Component {
  constructor (props) {
    super(props);
    this.state = {
      number: 0
    }
  }
  handleClick = () => {
    this.setState({number: this.state.number + 1})
  }
  render() {
    return (
      <>
        <div>{this.state.number}</div>
        <button onClick={this.handleClick}>+</button>
      </>
    )
  }
}

class App extends React.Component {
  constructor (props) {
    super(props);
    this.state = {
      number: 0
    }
  }
  handleClick = () => {
    this.setState({number: this.state.number + 1})
  }
  render() {
    let MyComponent = WidthWrapper(Example) // 在render中创建
    return (
      <>
        <div>{this.state.number}</div>
        <button onClick={this.handleClick}>+</button>
        <MyComponent />
      </>
    )
  }
}

export default App;
```

重新挂载组件会导致该组件及其所有子组件的状态丢失。

解决上面的问题
```js
function WidthWrapper (WrappedComponent) {
  return class extends React.Component {
    componentDidMount() {
      console.log('页面加载完成')
    }
    render() {
      return (
        <WrappedComponent />
      );
    }
  }
}

class Example extends React.Component {
  constructor (props) {
    super(props);
    this.state = {
      number: 0
    }
  }
  handleClick = () => {
    this.setState({number: this.state.number + 1})
  }
  render() {
    return (
      <>
        <div>{this.state.number}</div>
        <button onClick={this.handleClick}>+</button>
      </>
    )
  }
}

let MyComponent = WidthWrapper(Example) // 在组件之外创建

class App extends React.Component {
  constructor (props) {
    super(props);
    this.state = {
      number: 0
    }
  }
  handleClick = () => {
    this.setState({number: this.state.number + 1})
  }
  render() {
    return (
      <>
        <div>{this.state.number}</div>
        <button onClick={this.handleClick}>+</button>
        <MyComponent />
      </>
    )
  }
}

export default App;
```

如果在组件之外创建 HOC，这样一来组件只会创建一次。因此，每次 render 时都会是同一个组件。


```js
function WidthWrapper (WrappedComponent) {
  return class extends React.Component {
    componentDidMount() {
      console.log('页面加载完成')
    }
    render() {
      return (
        <WrappedComponent />
      );
    }
  }
}

class Example extends React.Component {
  constructor (props) {
    super(props);
    this.state = {
      number: 0
    }
  }
  handleClick = () => {
    this.setState({number: this.state.number + 1})
  }
  render() {
    return (
      <>
        <div>{this.state.number}</div>
        <button onClick={this.handleClick}>+</button>
      </>
    )
  }
}


class App extends React.Component {
  constructor (props) {
    super(props);
    this.state = {
      number: 0
    }
    this.myComponent = WidthWrapper(Example) // 在构造函数里面创建
  }
  handleClick = () => {
    this.setState({number: this.state.number + 1})
  }
  render() {
    let MyComponent = this.myComponent

    return (
      <>
        <div>{this.state.number}</div>
        <button onClick={this.handleClick}>+</button>
        <MyComponent />
      </>
    )
  }
}
```

在极少数情况下，你需要动态调用HOC。你可以在组件的生命周期方法或其构造函数中进行调用。
