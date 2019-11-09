---
title: Jest单元测试
date: 2019-11-09 10:08:38
tags: Jest
---
{% asset_img jest.png 图片 %}

单元测试主要包含断言、测试框架、测试用例、测试覆盖率、mock、持续集成等几个方面，由于Node的特殊性，它还会加入异步代码测试和私有方法的测试这两个部分。

<!-- more -->

了解单元测试之前，先回顾一下之前我们是怎样测试代码的。下面我写了一个求和函数。


```js
function sum (a, b) {
  return a + b
}
// 通过console输出值跟预期的结果对比
console.log(sum(1, 2), 3)
```


通常是在开发环境console输出结果，然后在线上环境把这个console删除掉，但是别人使用的时候又会再测一下这个功能是否正常。这样也太麻烦了。


这个时候就发现写单元测试的必要性了，而且单元测试可以提高我们的代码质量，更好的理解代码的功能。



现在主流的测试框架有Jest，Jest是facebook推出的一款测试框架，集成了 Mocha，chai，jsdom，sinon等功能。今天讲解的就是使用Jest进行单元测试。



安装jest和@types/jest包含的声明文件，声明文件是定义了一些类型，写代码的时候会有提示，如果你熟悉TS，大概明白这个文件了。


```
npm i jest @types/jest
```


然后加package.json添加一段脚本，作用是可以通过命令运行npm run test


```
"scripts": {
    "test": "jest"  
}
```


默认会测试文件名包含spec和test结尾的js文件

{% asset_img file.png 图片 %}


Jest默认只能测试模块，我现在把我index.js文件写的方法导出。

```js
export function sum (a, b) {
  return a + b
}
```


qs.sepc.js中测试用例的写法
```js
import { sum } from '../index'

it('测试sum函数', () => {
  expect(sum(1, 2)).toBe(3)
})
```


然后运行代码npm run test，生成测试报告

{% asset_img report.png 图片 %}
