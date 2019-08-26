---
title: 设计模式之装饰器模式
date: 2019-08-26 22:47:03
tags: JavaScript
---

{% asset_img banner.jpg 图片 %}

之前掌握的适配器模式是在原有结构基础上创建一个适配器，将原接口转换为客户希望的另一个接口，（只是为了兼容，不增加新功能）客户只需要和适配器打交道。

<!-- more -->

适配器模式主要用来解决两个已有接口之间不匹配的问题，它不考虑这些接口是怎样实
现的，也不考虑它们将来可能会如何演化。适配器模式不需要改变已有的接口，就能够
使它们协同作用。

装饰器模式也不会改变原有对象的接口，但装饰者模式的作用是为了给对象增加功能。

如果你了解AOP面向切片编程的话，那么理解装饰器就更容易了，因为它就是装饰器模式的应用。

下面after和before就是AOP的应用形式，具体代码如下：
```js
Function.prototype.before = function (beforefn) {
  var __self = this;
  return function () {
    beforefn.apply( this, arguments );
    return __self.apply(this, arguments)
  }
}
Function.prototype.after = function (afterfn) {
  var __self = this;
  return function () {
    var ret = __self.apply(this, arguments);
    afterfn.apply(this, arguments);
    return ret;
  }
};
```
用 AOP装饰函数的技巧在实际开发中非常有用。不论是业务代码的编写，还是在框架层面，我们都可以把行为依照职责分成粒度更细的函数，随后通过装饰把它们合并到一起，这有助于我们编写一个松耦合和高复用性的系统。

比如我们可以表单提交之前验证一下表单内容的合法性：
```js
function validata () {
  if (!this.value) {
    console.log('用户名不能为空')
  }
}
function submit () {
  console.log('提交')
}
let formSubmit = submit.before(validata)
```
校验输入和提交表单的代码完全分离开来，它们不再有任何耦合关系。
