---
title: 工厂模式
date: 2019-07-27 22:48:21
tags: JavaScript
---

{% asset_img banner.png 图片 %}

虽然 Object 构造函数或对象字面量都可以用来创建单个对象，但这些方式有个明显
的缺点：使用同一个接口创建很多对象，会产生大量的重复代码。

<!-- more -->

```js
let nan = {
  name: '张三',
  age: 18,
  like: 'js'
}
let woman = {
  name: '李四',
  age: 18,
  like: 'css'
}
```

为解决这个问题，人们开始使用工厂模式。

工厂模式是软件工程领域一种广为人知的设计模式，这种模式抽象了创建具体对象
的过程。

```js
function createPerson(name, age, like) {
  let obj = {};
  obj.name = name;
  obj.age = age;
  obj.like = like;
  return obj;
}
let nan = createPerson('张三', '18', 'js');
let woman = createPerson('李四', '18', 'css');
```

函数 `createPerson()` 能够根据接受的参数来构建一个包含所有必要信息的 `Person` 对
象。可以无数次地调用这个函数，而每次它都会返回一个包含三个属性一个方法的对象。

工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问
题（即怎样知道一个对象的类型）。随着JavaScript的发展，又一个新模式出现了。

简单工厂模式是由一个工厂对象决定创建出哪一种产品类的实例。

```js
class Person {
  constructor(name, age, like) {
    this.name = name;
    this.age = age;
    this.like = like;
  }
}

class Man extends Person {
  // TODO
}

class Woman extends Person {
  // TODO
}

class Factory {
  static create(type) {
    switch (type) {
      case 'man':
        return new Man('张三', 18, 'js')
      case 'woman':
        return new Woman('李四', 18, 'css')
      default:
        throw new Error('其他类型')
    }
  }
}

let p1 = Factory.create('man')
let p2 = Factory.create('woman')
```

当需要根据不同参数产生不同实例，这些实例都有相同的行为，这时候我们可以使用工厂模式，简化实现的过程，同时也可以减少每种对象所需的代码量。工厂模式有利于消除对象间的耦合，提供更大的灵活性。
