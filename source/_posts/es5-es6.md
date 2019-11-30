---
title: ES5和ES6类的知识
date: 2019-11-30 11:55:16
tags: JavaScript
---
{% asset_img banner.png 图片 %}

ECMAScript中的构造函数可用来创建特定类型的对象。像 Object 和 Array 这样的原生构造函数，在运行时会自动出现在执行环境中。此外，也可以创建自定义的构造函数，从而定义自定义对象类型的属性和方法。

<!-- more -->


```js
function Person(name, age, job) {
  this.name = name
  this.age = age
  this.job = job
  this.sayName = function () {
    console.log(this.name)
  }
}
const person1 = new Person('张三', 18, 'student')
```


要创建 Person 的新实例，必须使用 new 操作符。以这种方式调用构造函数实际上会经历以下4个步骤：


1. 创建一个新对象
2. 将构造函数的作用域赋给新对象（因此 this 就指向了这个新对象）
3. 执行构造函数中的代码（为这个新对象添加属性）
4. 返回新对象



ES6 提供了更接近传统语言的写法，引入了 Class（类）这个概念，作为对象的模板。通过class关键字，可以定义类。


```js
class Person {
  constructor(name, age, job) {
    this.name = name
    this.age = age
    this.job = job
    this.sayName = function () {
      console.log(this.name)
    }
  }
}
const person1 = new Person('张三', 18, 'student')
```


上面代码定义了一个“类”，可以看到里面有一个constructor方法，这就是构造方法，而this关键字则代表实例对象。也就是说，ES5 的构造函数Person，对应 ES6 的Person类的构造方法constructor。


```js
class Person {
  // ...
}

typeof Person // "function"

Person === Person.prototype.constructor // true
```


上面代码表明，类的数据类型就是函数，类本身就指向构造函数。使用的时候，也是直接对类使用new命令，跟构造函数的用法完全一致。



构造函数的prototype属性，在 ES6 的“类”上面继续存在。事实上，类的所有方法都定义在类的prototype属性上面。


```js
class Person {
  constructor () {
    // ...
  }
  eat () {
    console.log('food')
  }
}

// 等同于
```

```js
Person.prototype = {
  constructor () {},
  eat () {
    console.log('food')
  }
}
```





另外，ES6 类的内部所有定义的方法，都是不可枚举的。可以被for-in循环遍历出来的属性称为枚举属性。


```js
for (const key in person1) {
  console.log(key)
}
```


但是 ES5 类的内部定义的方法是可以枚举的。
```js
function Person() {
  // ...
}
const person1 = new Person()

Person.prototype.eat = function () {
  console.log('eat')
}

for (const key in person1) {
  console.log(key) // eat
}
```


如果只想遍历实例上的私有属性，需要添加额外的代码判断逻辑。通过hasOwnProperty可以做到。
```js
for (const key in person1) {
  if (person1.hasOwnProperty(key)) {
    // ....
  }
}
```
