---
title: 深入浅出测试用例
date: 2019-10-20 20:07:50
tags: Jest
---
{% asset_img banner.png 图片 %}

之前的文章我们已经讲解了Jest单元测试的基本使用流程，还有一些比较复杂的概念将会在这篇文章中介绍，包含怎样测试异步方法，怎样mock接口数据，怎样生成测试报告，等等

<!-- more -->



## 测试风格



我们将测试用例的不同组织方式称为测试风格，现今流行的单元测试风格主要有TDD（测试驱动开发）和BDD（行为驱动开发）两种。



关注点不同。TDD关注所有功能是否被正确实现，每一个功能都具备对应的测试用例；BDD关注整体行为是否符合预期，适合自顶向下的设计方式。




我们一般会选择BDD形式的测试风格，先开发代码，再测试最后结果是否符合预期。




### 测试异步


写异步代码通常会使用回调和promise的形式，那么我们把这两种形式的代码都进行测试。


```js
// 回调
export const getDataCallback = fn => {
  setInterval(() => {
    fn({name: 'callback'})
  }, 1000);
}

// promise
export const getDataPromise = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({name: 'promise'})
    }, 1000);
  })
}
```


Jest对异步代码的测试有三种方式。第一种是调用回调函数done，第二种是返回promise，第三种是使用高级语法asyn函数。


```js
describe('测试异步方法', () => {
  // 调用回调done标识完成
  it('测试回调函数', (done) => {
    getDataCallback((data) => {
      expect(data).toEqual({ name: 'callback' })
      done()
    })
  })

  it('测试promise', () => {
    // 返回的promise会等待完成
    return getDataPromise().then(data => {
      expect(data).toEqual({ name: 'promise' })
    })
  })

  it('测试promise', async () => {
    // 使用async、await语法
    const data = await getDataPromise()
    expect(data).toEqual({ name: 'promise' })
  })
})
```


我们再次回顾单元测试的语法规则，可以使用describe将测试用例进行分组，it表示单个的测试用例，一个功能至少包含一个测试用例，而且最好应该包含正面和反面两种情况进行测试，也就是相等和不相等的情况。expect表示预期的结果，toEqual称为匹配器，是比较对象的属性是否相等，如果相等单元测试就可以通过。


```js
it('测试不相等', () => {
  expect(1+1).not.toBe(3) // 1+1不等3
  expect(3).toBeLessThan(5) // 3<5
})

it('测试包含', () => {
  expect('hello').toContain('h') // 包含字符串h
  expect('hello').toMatch(/h/) // 参数可以是正则
})
```

Jest内置了Jsdom模拟了浏览器的环境，所有我们可以测试一些DOM操作的代码。


```js
it('测试删除DOM', () => {
  document.body.innerHTML = `<div><button></button></div>`

  let button = document.querySelector('button')
  expect(button).not.toBe(null)

  // 自己写的移除的DOM方法
  removeNode(button);
  button = document.querySelector('button')
  expect(button).toBe(null)
})
```


## mock数据


如果我们要测试接口，比如使用axios发送api请求，代码如下。
```js
import axios from 'axios'

// 测试接口调用
export const fetchUser = () => {
  return axios.get('/list')
}
```


我们并不会真实的发送API请求，而是模拟数据返回，这个时候可以在`__mocks__`下面建axios.js文件，jest就不会再使用node_modules文件中的axios模块了，而是使用我们自己写的axios模块。


```js
// mock axios
export default {
  get(url) {
    return new Promise((resolve, reject) => {
      // 根据请求路径mock数据
      if (url === '/list') {
        resolve({ name: 'hello' })
      }
    })
  }
}

import { fetchUser } from '../api'

it('测试请求列表', async () => {
  // 把请求的逻辑mock
  const data = await fetchUser()
  expect(data).toEqual({name: 'hello'})
})
```


## 测试覆盖率


想要查看测试覆盖率，我们需要先生成配置文件，通过命令。
```
npx jest --init
```


然后还需要在package.json中添加一段脚本。
```js
{
  "scripts": {
    "test": "jest --coverage"
  }
}
```


通过命令运行`npm run test`最后会生成测试覆盖率。


还会在coverage目录的lcov-report目录里生成HTML文件，可以通过浏览器打开查看。

{% asset_img coverage.png 图片 %}
