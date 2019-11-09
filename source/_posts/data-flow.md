---
title: 单向数据流
date: 2019-11-09 10:57:59
tags: Vue
---
{% asset_img banner.png 图片 %}

所有的 prop 都使得其父子 prop 之间形成了一个单向下行绑定：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。

<!-- more -->

vue VS react


vue和react都是单向数据流，但是当从子组件意外改变父级组件的状态时，vue可以更新视图，react不能。因为数据绑定的原理不同导致。







## vue





在Vue项目中子组件修改父组件的状态
```js
<template>
  <div id="app">
    <HelloWorld :count="count" />
  </div>
</template>
<script>
import HelloWorld from './HelloWorld.vue'
export default {
  components: { HelloWorld },
  data () {
    return {
      count:{ a: 1, b: 2 }
    }
  }
}
</script>
<template>
  <div class="hello">
    <div @click="handleClick">
        {{count.a}}
    </div>
  </div>
</template>
<script>
export default {
  props: {
    count: {
      type: Object,
      default: () => ({})
    }
  },
  methods: {
    handleClick () {
      this.count.a++
    }
  }
}
</script>
```


注意在 JavaScript 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变这个对象或数组本身将会影响到父组件的状态。





## react





在React项目中子组件修改父组件的状态
```js
 function Total(props) {
  const add = () => {
    props.count.a++
  }
  return (
    <div onClick={add}>
        {props.count.a}
    </div>)
}

function App() {
  const count = { a: 1, b: 2 }
  return (
    <div className="App">
      <Total count={count} />
    </div>
  );
}
```


所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。更新状态必须调用setState方法，属性是无法通过子组件更新的。
