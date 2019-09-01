---
title: 设计模式之原型模式
date: 2019-09-01 12:13:56
tags: JavaScript
---
{% asset_img banner.png 图片 %}

使用构造函数的主要问题，就是每个方法都要在每个实例中重新创建一遍。

<!-- more -->

```js
function Animal () {
  this.sayName = function () {
    console.log('动物')
  }
}

let dog = new Animal()
let cat = new Animal()
```
每个函数都有一个 `prototype` （原型）属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。

如果按照字面意思来理解，那么 `prototype` 就是通过调用构造函数而创建的那个对象实例的原型对象。


使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。
```js
function Animal () {
}

Animal.prototype.sayName = function () {
  console.log('动物')
}

let dog = new Animal()
let cat = new Animal()
```
要理解原型模式的工作原理，必须先理解原型对象的性质。

## 理解原型对象
无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一
个 `prototype` 属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个 `constructor` （构造函数）属性，这个属性包含一个指向 `prototype` 属性所在函数的指针。
```js
Animal.prototype.constructor === Animal
```
创建了自定义的构造函数之后，其原型对象默认只会取得 `constructor` 属性；除了原型上自定义的方法，还有一些其他其他方法，则都是从 Object 继承而来的。
{% asset_img p0.png 图片 %}

```js
Animal.prototype.__proto__ === Object.prototype
```

当调用构造函数创建一个新实例后，该实例的内部将包含一个指针，指向构造函数的原型对象。这个指针叫`__proto__ `。
{% asset_img p1.png 图片 %}

```js
dog.__proto__ === Animal.prototype
```


`Object.create(null)`创建的对象不会继承 Object 这个类，所有没有`__proto__`属性。

函数有`prototype`和`__proto__`属性，因为函数也是对象的一种，对象有`__proto__`属性，指向所属类的原型。
{% asset_img prototype.png 图片 %}
```js
console.log(Object instanceof Function);
console.log(Function instanceof Object);
console.log(Function instanceof Function);
```
原型的优势是可以随意扩展和重写继承的方法。
