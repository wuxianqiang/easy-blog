---
title: 设计模式之状态模式
date: 2019-09-01 12:17:51
tags: JavaScript
---
{% asset_img banner.png 图片 %}

状态模式是一种非同寻常的优秀模式，它也许是解决某些需求场景的最好方法。虽然状态模式并不是一种简单到一目了然的模式，但你一旦明白了状态模式的精髓，以后一定会感谢它带给你的无与伦比的好处。

<!-- more -->

状态模式的关键是区分事物内部的状态，事物内部状态的改变往往会带来事物的行为改变。

我们来想象一个场景，路口的红绿灯。它有三种状态，可以不断的轮询切换。
{% asset_img light.jpg 图片 %}

根据这个交通规则，我们通过代码的形式来描述。

```js
class Light {
  constructor () {
    this.currentState = '红灯'
  }
  change () {
    if (this.currentState === '红灯') {
      console.log('显示红灯')
      this.currentState = '绿灯'
    } else if (this.currentState === '绿灯') {
      console.log('显示绿灯')
      this.currentState = '红灯'
    }
  }
}

const light = new Light()
light.change()
```
通过先用一个变量 `currentState` 来记录路灯的当前状态，在事件发生时，再根据这个状态来决定下一步的行为。这里是先显示红灯再显示绿灯这样依次轮询。

现在使用状态模式改进电灯的程序。状态模式的关键是把事物的每种状态都封装成单独的类，跟此种状态有关的行为都被封装在这个类的内部，所以每次路灯改变的时候，把这个请求委托给当前的状态对象即可，该状态对象会负责渲染它自身的行为。
```js
class RedLight  {
  constructor (light) {
    this.light = light;
  }
  show () {
    console.log('显示红灯')
    this.light.setState(this.light.greenLight)
  }
}

class GreenLight  {
  constructor (light) {
    this.light = light;
  }
  show () {
    console.log('显示绿灯')
    this.light.setState(this.light.redLight)
  }
}

class Light {
  constructor () {
    this.redLight = new RedLight(this)
    this.greenLight = new GreenLight(this)
    this.currentState = this.redLight
  }
  setState (newState) {
    this.currentState = newState
  }
  change () {
    this.currentState.show()
  }
}

const light = new Light()
light.change()

```
现在不再使用一个字符串来记录当前的状态（红灯，绿灯），而是使用更加立体化的状态对象（`RedLight`，`GreenLight`）。我们在 `Light` 类的构造函数里为每个状态类都创建一个状态对象，这样一来我们可以很明显地看到电灯一共有多少种状态。

最后还要提供一个 `setState` 方法，状态对象可以通过这个方法来切换 `currentState` 对象的状态。状态的切换规律事先被完好定义在各个状态类中。有效的减少了 `if` 分支语句的写法，代码更加优雅整洁。

除了使用状态模式可以减少分支语句，还可以通过 Generator 函数实现。

Generator 是实现状态机的最佳结构。
```js
let state = function* () {
  while (1) {
    yield "A";
    yield "B";
    yield "c";
  }
}
// 在三种状态中不断切换
var obj = state();
console.log(obj.next()) //{ value: 'A', done: false }
console.log(obj.next()) //{ value: 'B', done: false }
console.log(obj.next()) //{ value: 'C', done: false }
console.log(obj.next()) //{ value: 'A', done: false }
```
每运行一次，就改变一次状态。上面代码是不断的切换A、B、C，三种状态。

如果业务复杂还是建议大家使用状态模式拆分代码，这样方便以后拓展。
