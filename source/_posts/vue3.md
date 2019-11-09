---
title: Vue2和Vue3响应式原理
date: 2019-10-20 20:07:07
tags: Vue
---
{% asset_img banner.png 图片 %}

Vue3.0在国庆节的时候突然更新，让我们这群开发者又踏上了学习Vue的路上，更新的内容有人说好，又有人说不好，其实这并不重要，重要的是API变化太大了。

<!-- more -->

之前Vue数据响应式是使用了defineProperty的特性，我们再次回顾defineProperty进行数据劫持。


```js
function observer (target) {
  if (typeof target !== 'object' && target === null) {
    return target
  }
  for (let key in target) {
    defineReactive(target, key, target[key])
  }
}

function defineReactive(target, key, value) {
  observer(value)
  Object.defineProperty(target, key, {
    get () {
      return value
    },
    set (newValue) {
      if (newValue !== value) {
        observer(newValue)
        value = newValue
      }
    }
  })
}

let obj = {a: 1}
observer(obj)
```


Vue把对象中的所有属性都通过defineProperty定义，而且每当对象设置值的时候会触发视图的更新，但是Vue也有不能触发视图的更新的几种情况，第一个就是不存在的属性不能触发更新，直接修改数组不能触发更新，这些就是缺点所在了。






Vue3.0为了修复这些问题，使用proxy进行数据劫持，这是ES6的语法，性能十分优异。



下面是使用Vue3.0编写组件的示例代码
```html
<body>
  <div id="container"></div>
  <script src="vue.global.js"></script>

  <script>
    // 我们获取鼠标的位置
    function usePosition () {
      let position = Vue.reactive({x: 0, y: 0})
      function update (e) {
        position.x = e.pageX
        position.y = e.pageY
      }
      // 函数里面使用生命周期
      Vue.onMounted(() => {
        window.addEventListener('mousemove', update)
      })

      Vue.onUnmounted(() => {
        window.removeEventListener('mousemove', update)
      })
      return position
    }

    const App = {
      // 启动函数
      setup () {
        let state = Vue.reactive({name: 'hello world'})
        let position = usePosition() // 这里调用了鼠标位置
        // 返回的对象会作为渲染的上下文
        return {
          state,
          change,
          position
        }
        function change () {
          // 这是普通函数，this不是实例
          state.name = '前端精髓'
        }
      },
      template: `<div @click="change">{{state.name}}</div>`
    }
    Vue.createApp().mount(App, container)
  </script>
</body>
```


vue3.0为了更好的类型推断，（避免使用装饰器），完全使用普通函数，用TS重写了源码，所以在项目中你使用Vue3.0开发，最好和TS搭配使用。



数据通信之前使用高阶组件（会导致没有更新的组件也进行重新渲染），mixin（变量名会和组件内部冲突）,作用域插槽（会导致数据来源不明确），现在Vue开发了一个@vue/composition-api组合式API，让组件的复用变得更加简单。



Vue3.0中通过reactive定义响应式的数据。
```js
let proxy = Vue.reactive({ name: 'hello world' })
```


我们可以演示proxy代理对象的过程
```js
function reactive(target) {
  return createReactiveObject(target)
}
function isObject(value) {
  return typeof value === 'object' && value !== null
}

// 弱引用映射表，防止对象被重复代理
let toProxy = new WeakMap(); // 源对象：代理过的对象
let toRaw = new WeakMap(); // 代理过的对象：源对象

// 创建响应式的对象
function createReactiveObject(target) {
  if (!isObject(target)) {
    return target
  }
  let proxy = toProxy.get(target) // 是不是已经代理过了
  if (proxy) {
    return proxy
  }
  if (toRaw.has(target)) {
    // 通过代理的对象再次代理，直接返回
    return target
  }

  // 拦截属性
  let baseHandler = {
    get(target, key, receiver) {
      let result = Reflect.get(target, key, receiver)
      // 取值的时候会递归
      return isObject(result) ? reactive(result) : result 
    },
    set(target, key, value, receiver) {
      let result = Reflect.set(target, key, value, receiver)
      return result
    },
    deleteProperty(target, key) {
      let result = Reflect.deleteProperty(target, key)
      return result
    }
  }

  let observed = new Proxy(target, baseHandler)
  toProxy.set(target, observed)
  toRaw.set(observed, target)
  return observed
}
```


Vue3.0的语法目前还是预览版本，但是API已经基本确定下来了，内部源码还可能发生改动，不建议在线上环境进行使用。
