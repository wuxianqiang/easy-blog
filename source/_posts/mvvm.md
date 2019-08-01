---
title: 分分钟写出Vue原理
date: 2019-08-01 08:16:29
tags: Vue
---

{% asset_img banner.png 图片 %}

<!-- more -->

第一步完成深度观察
```js
class Vue {
  constructor (options) {
    this.$el = options.el;
    this.$options = options;
    this.$data = options.data;
    if (this.$el) {
      new Observer(this.$data)
    }
  }
}

class Observer {
  constructor(data) {
    this.observe(data)
  }
  observe (obj) {
    if (obj && typeof obj === 'object') {
      for (const key in obj) {
        this.defineReactive(obj, key, obj[key])
      }
    }
  }
  defineReactive (obj, key, value) {
    this.observe(value)
    Object.defineProperty(obj, key, {
      get () {
        return value
      },
      set: (newValue) => {
        if (value !== newValue) {
          value = newValue
          this.observe(newValue)
        }
      }
    })
  }
}
```
第二步将DOM放到内存中
```js
class Vue {
  constructor (options) {
    this.$el = options.el;
    this.$options = options;
    this.$data = options.data;
    if (this.$el) {
      new Observer(this.$data)
      new Compiler(this.$el, this)
    }
  }
}

class Compiler {
  constructor (el, vm) {
    this.vm = vm;
    this.el = this.isElementNode(el) ? el : document.querySelector(el);
    let fragment = this.nodeFragment(this.el);
    this.compile(fragment)
    this.el.appendChild(fragment)
  }
  compile (node) {
    console.log(node, '分析DOM节点')
  }
  nodeFragment (node) {
    let fragment = document.createDocumentFragment()
    let firstChild;
    while (firstChild = node.firstChild) {
      fragment.appendChild(firstChild)
    }
    return fragment
  }
  isElementNode (node) {
    return node.nodeType === 1;
  }
}
```
第三步分析模板里面的内容
```js
let CompileUtil = {
  text (node, expr, vm) {
    let handler = this.updater['text']
    let content = expr.replace(/\{\{(.+?)\}\}/, (...args) => {
      return this.getValue(args[1], vm)
    })
    handler(node, content)
  },
  getValue(expr, vm) {
    return expr.split('.').reduce((data, current) => {
      return data[current]
    }, vm.$data)
  },
  updater: {
    text (node, value) {
      node.textContent = value
    }
  }
}

class Compiler {
  constructor (el, vm) {
    this.vm = vm;
    this.el = this.isElementNode(el) ? el : document.querySelector(el);
    let fragment = this.nodeFragment(this.el);
    this.compile(fragment)
    this.el.appendChild(fragment)
  }
  compile (node) {
    let nodeList = node.childNodes;
    Array.from(nodeList, child => {
      if (this.isElementNode(child)) {
        this.compileElement(child)
        this.compile(child)
      } else {
        this.compileText(child)
      }
    })
  }
  compileElement (node) {
    // 分析元素节点
  }
  compileText (node) {
    // 分析文本节点
    // 1. 小胡子语法里面要替换成我们想要的内容
    let content = node.textContent;
    if (/\{\{(.+?)\}\}/.test(content)) {
      CompileUtil.text(node, content, this.vm)
    }
  }
  nodeFragment (node) {
    let fragment = document.createDocumentFragment()
    let firstChild;
    while (firstChild = node.firstChild) {
      fragment.appendChild(firstChild)
    }
    return fragment
  }
  isElementNode (node) {
    return node.nodeType === 1;
  }
}
```
第三步观察者模式进行绑定
```js
class Dep {
  constructor() {
    this.subs = []
  }
  addSub(watcher) {
    this.subs.push(watcher)
  }
  notify() {
    this.subs.forEach(watcher => watcher.update())
  }
}

class Watcher {
  constructor(vm, expr, cb) {
    this.vm = vm;
    this.expr = expr;
    this.cb = cb;
    this.oldVale = this.get()
  }
  get () {
    Dep.target = this;
    // 取值操作
    let value = CompileUtil.getValue(this.expr, this.vm)
    Dep.target = null;
    return value
  }
  update () {
    let newValue = CompileUtil.getValue(this.expr, this.vm)
    if (newValue !== this.oldVale) {
      // 当比较两个值不相等的情况下执行回调
      this.cb(this.newValue)
    }
  }
}

class Vue {
  constructor(options) {
    this.$el = options.el;
    this.$options = options;
    this.$data = options.data;
    if (this.$el) {
      new Observer(this.$data)
      new Compiler(this.$el, this)
    }
  }
}

class Observer {
  constructor(data) {
    this.observe(data)
  }
  observe(obj) {
    if (obj && typeof obj === 'object') {
      for (const key in obj) {
        this.defineReactive(obj, key, obj[key])
      }
    }
  }
  defineReactive(obj, key, value) {
    this.observe(value)
    let dep = new Dep()
    Object.defineProperty(obj, key, {
      get() {
        Dep.target && dep.addSub(Dep.target)
        return value
      },
      set: (newValue) => {
        if (value !== newValue) {
          value = newValue
          this.observe(newValue)
          dep.notify()
        }
      }
    })
  }
}

let CompileUtil = {
  text(node, expr, vm) {
    let handler = this.updater['text']
    let content = expr.replace(/\{\{(.+?)\}\}/, (...args) => {
      new Watcher(vm, args[1], () => {
        // 处理函数
        handler(node, this.getContentValue(vm, expr))
      })
      return this.getValue(args[1], vm)
    })
    handler(node, content)
  },
  getContentValue (vm, expr) {
    return expr.replace(/\{\{(.+?)\}\}/, (...args) => {
      return this.getValue(args[1], vm)
    })
  },
  getValue(expr, vm) {
    return expr.split('.').reduce((data, current) => {
      return data[current]
    }, vm.$data)
  },
  updater: {
    text(node, value) {
      node.textContent = value
    }
  }
}

class Compiler {
  constructor(el, vm) {
    this.vm = vm;
    this.el = this.isElementNode(el) ? el : document.querySelector(el);
    let fragment = this.nodeFragment(this.el);
    this.compile(fragment)
    this.el.appendChild(fragment)
  }
  compile(node) {
    let nodeList = node.childNodes;
    Array.from(nodeList, child => {
      if (this.isElementNode(child)) {
        this.compileElement(child)
        this.compile(child)
      } else {
        this.compileText(child)
      }
    })
  }
  compileElement(node) {
    // 分析元素节点
  }
  compileText(node) {
    // 分析文本节点
    // 1. 小胡子语法里面要替换成我们想要的内容
    let content = node.textContent;
    if (/\{\{(.+?)\}\}/.test(content)) {
      CompileUtil.text(node, content, this.vm)
    }
  }
  nodeFragment(node) {
    let fragment = document.createDocumentFragment()
    let firstChild;
    while (firstChild = node.firstChild) {
      fragment.appendChild(firstChild)
    }
    return fragment
  }
  isElementNode(node) {
    return node.nodeType === 1;
  }
}
```
