---
title: 设计模式之单例模式
date: 2019-08-25 09:48:27
tags: JavaScript
---

{% asset_img banner.png 图片 %}

在之前我们已经掌握了设计模式中的工厂模式，工厂模式就是用函数封装创建对象的具体过程来批量创建对象的方式。

<!-- more -->

今天说说设计模式中的单例模式，先明白什么是单例模式？单例模式指定的只有一个实例对象。

> 单例模式的定义是：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

单例模式是一种常用的模式，有一些对象我们往往只需要一个，比如线程池、全局的缓存、浏览器中的 window 对象等。


在 JavaScript开发中，单例模式的用途同样非常广泛。

试想一下，当我们单击登录按钮的时候，页面中会出现一个登录浮窗，而这个登录浮窗是唯一的，无论单击多少次登录按钮，这个浮窗都只会被创建一次，那么这个登录浮窗就适合用单例模式来创建。

```js
class Login {
  constructor() {
    this.element = document.createElement('div');
    this.element.innerHTML = '用户名 <input type="text"/><button>登录</button>'
    this.element.style.cssText = 'width: 100px; height: 100px; position: absolute; left: 50%; top: 50%; display: block;';
    document.body.appendChild(this.element);
  }
  show() {
    this.element.style.display = 'block';
  }
  hide() {
    this.element.style.display = 'none';
  }
}
Login.getInstance = (function () {
  let instance;
  return function () {
    if (!instance) {
      instance = new Login();
    }
    return instance;
  }
})();

document.getElementById('showBtn').addEventListener('click', function (event) {
  Login.getInstance().show();
});
document.getElementById('hideBtn').addEventListener('click', function (event) {
  Login.getInstance().hide();
});
```
## 实现单例模式

要实现一个标准的单例模式并不复杂，无非是用一个变量来标志当前是否已经为某个类创建过对象，如果是，则在下一次获取该类的实例时，直接返回之前创建的对象。代码如下：

```js
class Window {
  constructor (name) {
    this.name = name;
  }
  static getInstance () {
    if (!this.instance) {
      this.instance = new Window()
    }
    return this.instance;
  }
}

let w1 = Window.getInstance();
let w2 = Window.getInstance();
// 变量w1和w2相等，表示只有一个window实例
console.log(w1 === w2)
```
们通过 Window.getInstance 来获取 Window 类的唯一对象，这种方式相对简单，但有一个问题，就是增加了这个类的“不透明性”

Window 类的使用者必须知道这是一个单例类，跟以往通过 new XXX 的方式来获取对象不同，这里偏要使用 Window.getInstance 来获取对象。

## 透明的单例模式

改进之前的问题，我们现在的目标是实现一个“透明”的单例类，用户从这个类中创建对象的时候，可以像使用其他任何普通类一样的调用。
```js
let Window = (function () {
  let window;
  let Window = function (name) {
    if (window) {
      return window;
    } else {
      this.name = name;
      return (window = this);
    }
  }
  return Window;
})();

let w1 = new Window();
let w2 = new Window();
console.log(w1 === w2)
```
看明白这段代码，你必须了解`new`关键字的原理。

上面这段代码把创建实例的逻辑和区分单例的逻辑耦合到了一起，现在我们要解耦这段代码：
```js
function Window(name) {
  this.name = name;
}

Window.prototype.getName = function() {
  console.log(this.name)
}

let CreateSingle = (function () {
  let instance;
  return function (name) {
    if (!instance) {
      instance  = new Window(name);
    }
    return instance
  }
})()

let w1 = new CreateSingle();
let w2 = new CreateSingle();
console.log(w1 === w2)
```
单例模式是一种简单但非常实用的模式，并且只创建唯一的一个。
