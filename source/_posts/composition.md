---
title: 在Vue中为什么需要组合API
date: 2020-02-21 21:55:31
tags: Vue
---

{% asset_img banner.png 图片 %}

目前在Vue世界中，最热门的话题是Composition API，这是Vue 3.0中引入的基于函数的API。在本文中，我们将学习这个新API，然后学习如何使用基于函数的API。我也尝试以尽可能简单的方式解释它。

<!-- more -->

### 所以，让我们开始吧~



在我们深入探讨之前，无需担心组合API将彻底改变Vue。Composition API是现有组件附加的功能，因此Vue 3.0中没有重大更改，此外，Vue团队还创建了一个与Vue 2.x兼容的插件（vue-composition）。


### 我们先来看一些问题！


> 为什么需要Vue的组合API？


随着Vue的日益普及，人们也开始在大型和企业级应用程序中采用Vue。由于这种情况，在很多情况下，此类应用程序的组件会随着时间的推移而逐渐增长，并且有时使用单文件组件人们很难阅读和维护。因此，需要以逻辑方式制动组件，而使用Vue的现有API则不可能。


> Vue已有的替代方案缺点是什么？


在引入新的API之前，Vue还有其他替代方案，它们提供了诸如mixin，HOC（高阶组件），作用域插槽之类的，提高组件之间的复用性，但是所有方法都有其自身的缺点，由于它们未被广泛使用。


1. Mixins：一旦应用程序包含一定数量的mixins，就很难维护。开发人员需要访问每个mixin，以查看数据来自哪个mixin。
2. HOC：这种模式不适用于.vue 单文件组件
3. 作用域插槽：在组件模板中放置了越来越多的插槽，导致数据来源不明确。


简而言之，组合API有助于

1. 由于API是基于函数的，因此可以有效地组织和编写可复用的代码。
2. 通过将共享逻辑分离为功能来提高代码的可读性。
3. 实现代码分离。
4. 在Vue应用程序中更好地使用TypeScript。


### 让我们来使用API构建一个简单的应用程序。



1）导入vue-composition插件
```js
npm install @vue/composition-api --save
```

2）在导入任何其他API之前，将插件安装到Vue应用程序。需在其他任何插件之前进行注册，Composition API可以在其他插件中使用，就像它是Vue的内置功能一样。
```js
import VueCompositionApi from '@vue/composition-api';
Vue.use(VueCompositionApi);
```

3）使用组合API，让我们创建一个小的功能以更好地了解API。创建一个名为composition-fns的新文件夹，并创建一个名为 toggle.js 的文件


在这里，我们从API导入`ref`，并使用它声明变量 `isVisible`，其默认值为 `false`。除此之外，还有一种称为“切换”的方法，它负责用于切换 `isVisible` 的值。

```js
import { ref } from "@vue/composition-api";


export const useToggle = () => {
  const isVisible = ref(false);


  const toggle = () => {
    return (isVisible.value = !isVisible.value);
  };


  return {
    isVisible,
    toggle
  };
};
```

4）首先，导入useToggle 函数，然后在组件中使用，Composition API提供了 `setUp()` 函数。这个启动函数再组件渲染时会执行。
```js
import { useToggle } from "./composition-fn/toggle";


setup() {
  const { isVisible, toggle } = useToggle();
  // expose to template
  return {
    isVisible,
    toggle
  };
}
```

5）最后，模板由按钮和div组成，以显示和隐藏切换值。
```html
<div>{{ isVisible }}</div>


<button type="button" @click="toggle">
  {{isVisible ? 'hide' : 'show' }} Toggle
</button>
```


点“[阅读原文](https://codesandbox.io/s/composition-api-iztg4)”查看完整代码！

main.js
```js
import Vue from "vue";
import App from "./App.vue";
import VueCompositionApi from '@vue/composition-api';

Vue.use(VueCompositionApi);

Vue.config.productionTip = false;

new Vue({
  render: h => h(App)
}).$mount("#app");
```
app.vue
```js
<template>
  <div id="app">
    <div>{{ isVisible }}</div>
    <button type="button" @click="toggle">{{isVisible ? 'hide' : 'show' }} Toggle</button>
  </div>
</template>

<script>
import { useToggle } from "./composition-fn/toggle";
export default {
  name: "App",
  setup(props, context) {
    const { isVisible, toggle } = useToggle();
    // expose to template
    return {
      isVisible,
      toggle
    };
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```
composition-fn/toggle.js
```js
import { ref } from "@vue/composition-api";

export const useToggle = () => {
  const isVisible = ref(false);

  const toggle = () => {
    console.log(isVisible);
    return (isVisible.value = !isVisible.value);
  };

  return {
    isVisible,
    toggle
  };
};
```
