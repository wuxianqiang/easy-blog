---
title: 单元测试介绍
date: 2019-07-01 22:31:26
tags: Node
---

{% asset_img banner.jpg 图片 %}

单元测试主要包含断言、测试框架、测试用例、测试覆盖率、mock、持续集成等几个方面，由于Node的特殊性，它还会加入异步代码测试和私有方法的测试这两个部分。

<!-- more -->

## 断言

如果有对Node的源码进行过研究，会发现Node中存在着 assert 这个模块，以及很多主要模块都调用了这个模块。何谓断言，维基百科上的解释是：

在程序设计中，断言（assertion）是一种放在程序中的一阶逻辑（如一个结果为真或是假的逻辑判断式），目的是为了标示程序开发者预期的结果——当程序运行到断言的位置时，对应的断言应该为真。若断言不为真，程序会中止运行，并出现错误信息。

一言以蔽之，断言用于检查程序在运行时是否满足期望。

如下代码是 assert 模块的工作方式：


```js
const assert = require('assert');
assert.strictEqual(Math.max(1, 100), 100);
```

一旦 `assert.strictEqual()` 不满足期望，将会抛出 `AssertionError` 异常，整个程序将会停止执行。没有对输出结果做任何断言检查的代码，都不是测试代码。

常见的断言方法

```
assert.deepStrictEqual(actual, expected)
```

测试 `actual` 参数和 `expected` 参数之间的深度相等。深度相等意味着子对象的可枚举的自身属性也通过以下规则进行递归计算。

```
assert.ok(value)
```
测试 value 是否为真值。等同于 `assert.strictEqual(!!value, true)`。


```
assert.strictEqual(actual, expected)
```
测试 `actual` 参数和 `expected` 参数之间的严格相等性


```
assert.notStrictEqual(actual, expected)
```
测试 `actual` 参数和 `expected` 参数之间的严格不相等

```js
assert.notDeepStrictEqual(actual, expected)
```

测试深度严格的不平等。与 `assert.deepStrictEqual()` 相反。



断言也有第三方模块，Chai 就是一个流行的断言库。因为它的语法比较语义化，所以很多人使用。

```js
// 相等或不相等
const expect = require('chai').expect;
expect(4 + 5).to.be.equal(9);
expect(4 + 5).to.be.not.equal(10);
expect(foo).to.be.deep.equal({ bar: 'baz' });
```

关于更多 Chai 断言语法，请阅读 Chai 官网[1]


## 测试框架

前面提到断言一旦检查失败，将会抛出异常停止整个应用，这对于做大规模断言检查时并不友好。更通用的做法是，记录下抛出的异常并继续执行，最后生成测试报告。这些任务的承担者就是测试框架。

测试框架用于为测试服务，它本身并不参与测试，主要用于管理测试用例和生成测试报告，提升测试用例的开发速度，提高测试用例的可维护性和可读性，以及一些周边性的工作。这里我们要介绍的优秀单元测试框架是 mocha。

## 测试风格

我们将测试用例的不同组织方式称为测试风格，现今流行的单元测试风格主要有TDD（测试驱动开发）和BDD（行为驱动开发）两种，它们的差别如下所示。

关注点不同。TDD关注所有功能是否被正确实现，每一个功能都具备对应的测试用例；BDD关注整体行为是否符合预期，适合自顶向下的设计方式。

表达方式不同。TDD的表述方式偏向于功能说明书的风格；BDD的表述方式更接近于自然语言的习惯。

mocha对于两种测试风格都有支持。

```js
const add = require('./add.js');
const expect = require('chai').expect;

describe('加法函数的测试', function() {
  it('1 加 1 应该等于 2', function() {
    expect(add(1, 1)).to.be.equal(2);
  });
});
```
关于更多 mocha 测试框架语法，请阅读 mocha 官网[2]

References
[1] Chai 官网: https://www.chaijs.com/guide/styles/
[2] mocha 官网: https://mochajs.org/#getting-started
