---
title: vue和react中props变化后修改state方式
date: 2019-08-17 18:36:18
tags: Vue
---

{% asset_img banner.png 图片 %}

<!-- more -->

如果只想在 `state` 更改时重新计算某些数据，比如搜索框案例。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190817105436149.png)
# vue
```js
<template>
  <div>
  	<input type="text" v-model="filterText">
    <ul>
      <li v-for="item in filteredList" :key="item.id">
        {{ item.text }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: {
    list: {
      type: Array,
      default: () => ([])
    }
  },
  data () {
    return {
      filterText: ''
    }
  },
  computed: {
    filteredList () {
      return this.list.filter(item => item.text.includes(this.filterText))
    }
  }
}
</script>
```
## react
```js
import React, { PureComponent } from 'react';

class Example extends PureComponent {
  state = {
    filterText: ''
  };
  handleChange = event => {
    this.setState({
      filterText: event.target.value
    })
  }
  render() {
    const filteredList = this.filter(this.props.list, this.state.filterText)
    return (
      <>
        <input
          type="text"
          onChange={this.handleChange}
          value={this.state.filterText} />
        <ul>
          {
            filteredList.map(
              item => <li key={item.id}>{item.text}</li>
            )
          }
        </ul>
      </>
    );
  }
}
```
如果你想在 `prop` 更改时“重置”某些 `state`，比如随机默认值案例
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190817113254717.png)

# vue
Vue提供了一种更通用的方式来观察和响应Vue实例上的数据变动：侦听属性 `watch`。
```js
<template>
  <div>
    <input type="text" v-model="text">
  </div>
</template>

<script>
export default {
  props: {
    email: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      text: ''
    }
  },
  watch: {
    email: {
      immediate: true,
      handler (value) {
        this.text = value
      }
    }
  }
}
</script>
```


# react
React生命周期 `getDerivedStateFromProps` 会在调用 `render` 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 `state`，如果返回 `null` 则不更新任何内容。

父组件重新渲染时触发，请注意，不管原因是什么，都会在每次渲染前触发此方法。

```js
class Example extends Component {
  state = {
    text: ''
  };
  handleChange = (event) => {
    this.setState({
      text: event.target.value
    })
  }
  static getDerivedStateFromProps(nextProps, prevState) {
    if (prevState.email !== nextProps.email) {
      return {
        text: nextProps.email,
        email: nextProps.email
      }
    }
    return {text: prevState.text}
  }
  render() {
    return (
      <>
        <input
          type="text"
          onChange={this.handleChange}
          value={this.state.text} />
      </>
    );
  }
}
```

# 改进

直接复制 `prop` 到 `state` 是一个非常糟糕的想法。这两者的关键在于，任何数据，都要保证只有一个数据来源，而且避免直接复制它。
## vue
```js
<template>
  <div>
    <input type="text" :value="value" @input="handleInput">
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: String,
      default: ''
    }
  },
  methods: {
    handleInput (e) {
      this.$emit('input', e.target.value)
    }
  }
}
</script>
```

```js
<template>
  <div id="app">
    <Example v-model="email"/>
    <button @click="handleClick">默认值</button>
  </div>
</template>

<script>
import Example from './components/Example.vue'

export default {
  components: {
    Example
  },
  data () {
    return {
      email: ''
    }
  },
  methods: {
    handleClick () {
      this.email = String(Math.random())
    }
  }
}
</script>
```
## react
```js
function Example (props) {
  return <input onChange={props.onChange} value={props.email} />;
}
```
```js
class App extends React.Component {
  state = {
    email: ''
  }
  handleClick = () => {
    this.setState({
      email: String(Math.random())
    })
  }
  handleChange = (event) => {
    this.setState({
      email: event.target.value
    })
  }
  render() {
    return (
      <>
        <Example email={this.state.email} onChange={this.handleChange} />
        <div>
          <button onClick={this.handleClick}>默认值</button>
        </div>
      </>
    );
  }
}
```
