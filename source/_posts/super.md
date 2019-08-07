---
title: 深入理解继承
date: 2019-08-07 20:54:50
tags: JavaScript
---

{% asset_img banner.png 图片 %}

<!-- more -->

## 原型链

其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

简单回顾一下构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。

那么，假如我们让原型对象等于另一个类型的实例，结果会怎么样呢？

```js
function SuperType(){
  this.property = true;
}
SuperType.prototype.getSuperValue = function(){
  return this.property;
};
function SubType(){
  this.subproperty = false;
}
//继承了SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function (){
  return this.subproperty;
};
var instance = new SubType();
alert(instance.getSuperValue()); //true
```

以上代码定义了两个类型： `SuperType` 和 `SubType` 。每个类型分别有一个属性和一个方法。它们的主要区别是 `SubType` 继承了 `SuperType` ，而继承是通过创建 `SuperType` 的实例，并将该实例赋给 `SubType.prototype` 实现的。实现的本质是重写原型对象，代之以一个新类型的实例。换句话说，原来存在于 `SuperType` 的实例中的所有属性和方法，现在也存在于 `SubType.prototype` 中了。

1. 原型链的问题

原先的实例属性也就顺理成章地变成了现在的原型属性了。

原型链的第二个问题是：在创建子类型的实例时，不能向超类型的构造函数中传递参数。实际上，应该说是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。

## 借用构造函数
在解决原型中包含引用类型值所带来问题的过程中，开发人员开始使用一种叫做借用构造函数的技术（有时候也叫做伪造对象或经典继承）。这种技术的基本思想相当简单，即在子类型构造函数的内部调用超类型构造函数。别忘了，函数只不过是在特定环境中执行代码的对象，因此通过使用 `apply()` 和 `call()` 方法也可以在（将来）新创建的对象上执行构造函数，如下所示：


```js
function SuperType(){
  this.colors = [“red”, “blue”, “green”];
}
function SubType(){
  //继承了SuperType
  SuperType.call(this);
}
var instance1 = new SubType();
instance1.colors.push(“black”);
alert(instance1.colors); //“red,blue,green,black”
var instance2 = new SubType();
alert(instance2.colors); //“red,blue,green”
```

通过使用 `call()` 方法（或 `apply()` 方法也可以），我们实际上是在（未来将要）新创建的 `SubType` 实例的环境下调用了 `SuperType` 构造函数。这样一来，就会在新 `SubType` 对象上执行 `SuperType()` 函数中定义的所有对象初始化代码。结果， `SubType` 的每个实例就都会具有自己的 `colors` 属性的副本了。

1. 传递参数

相对于原型链而言，借用构造函数有一个很大的优势，即可以在子类型构造函数中向超类型构造函数传递参数。

2. 借用构造函数的问题

如果仅仅是借用构造函数，那么也将无法避免构造函数模式存在的问题——方法都在构造函数中定义，因此函数复用就无从谈起了。而且，在超类型的原型中定义的方法，对子类型而言也是不可见的，结果所有类型都只能使用构造函数模式。考虑到这些问题，借用构造函数的技术也是很少单独使用的。

## 组合继承
组合继承，有时候也叫做伪经典继承，指的是将原型链和借用构造函数的技术组合到一块，从而发挥二者之长的一种继承模式。其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。


```js
function SuperType(name){
  this.name = name;
  this.colors = [“red”, “blue”, “green”];
}
SuperType.prototype.sayName = function(){
  alert(this.name);
};
function SubType(name, age){
  //继承属性
  SuperType.call(this, name);
  this.age = age;
}
//继承方法
SubType.prototype = new SuperType();
SubType.prototype.sayAge = function(){
  alert(this.age);
};
var instance1 = new SubType(“Nicholas”, 29);
instance1.colors.push(“black”);
alert(instance1.colors); //“red,blue,green,black”
instance1.sayName(); //“Nicholas”;
instance1.sayAge(); //29
var instance2 = new SubType(“Greg”, 27);
alert(instance2.colors); //“red,blue,green”
instance2.sayName(); //“Greg”;
instance2.sayAge(); //27
```

组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点，成为JavaScript中最常用的继承模式。而且， `instanceof` 和 `isPrototypeOf()` 也能够用于识别基于组合继承创建的对象。



## 原型式继承

他的想法是借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。
```js
function create(o){
  function F(){}
  F.prototype = o;
  return new F();
}
```
在 `create()` 函数内部，先创建了一个临时性的构造函数，然后将传入的对象作为这个构造函数的原型，最后返回了这个临时类型的一个新实例。

ECMAScript 5通过新增 `Object.create()` 方法规范化了原型式继承。这个方法接收两个参数：一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象。在传入一个参数的情况下， `Object.create()` 与 `create()` 方法的原理相同。


```js
let person = {
  name: 'Nicholas',
  friends: ['Shelby', 'Court', 'Van']
};
let anotherPerson = Object.create(person);
anotherPerson.name = 'Greg';
anotherPerson.friends.push('Rob');
let yetAnotherPerson = Object.create(person);
yetAnotherPerson.name = 'Linda';
yetAnotherPerson.friends.push('Barbie');
alert(person.friends); // 'Shelby,Court,Van,Rob,Barbie'
```
`Object.create()` 方法的第二个参数与 `Object.defineProperties()` 方法的第二个参数格式相同：每个属性都是通过自己的描述符定义的。以这种方式指定的任何属性都会覆盖原型对象上的同名属性。

```js
let person = {
  name: 'Nicholas',
  friends: ['Shelby', 'Court', 'Van']
};
let anotherPerson = Object.create(person, {
  name: {
    value: 'Greg'
  }
});
alert(anotherPerson.name); //'Greg'
```
## 寄生式继承

寄生式继承的思路与寄生构造函数和工厂模式类似，即创建一个仅用于封装继承过程的函数，该函数在内部以某种方式来增强对象，最后再像真地是它做了所有工作一样返回对象。以下代码示范了寄生式继承模式。

```js
function createAnother(original) {
  let clone = Object.assign({}, original); //通过调用函数创建一个新对象
  clone.sayHi = function () { //以某种方式来增强这个对象
    alert('hi');
  };
  return clone; //返回这个对象
}
```

## 寄生组合式继承

前面说过，组合继承是JavaScript最常用的继承模式；不过，它也有自己的不足。组合继承最大的问题就是无论什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。没错，子类型最终会包含超类型对象的全部实例属性，但我们不得不在调用子类型构造函数时重写这些属性。再来看一看下面组合继承的例子。

```js
function SuperType(name) {
  this.name = name;
  this.colors = ['red', 'blue', 'green'];
}
SuperType.prototype.sayName = function () {
  alert(this.name);
};
function SubType(name, age) {
  SuperType.call(this, name); //第二次调用SuperType()
  this.age = age;
}
SubType.prototype = new SuperType(); //第一次调用SuperType()
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function () {
  alert(this.age);
};
```

有两组 `name` 和 `colors` 属性：一组在实例上，一组在 `SubType` 原型中。这就是调用两次 `SuperType` 构造函数的结果。好在我们已经找到了解决这个问题方法——寄生组合式继承。

所谓寄生组合式继承，即通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。其背后的基本思路是：不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型原型的一个副本而已。本质上，就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型。寄生组合式继承的基本模式如下所示。
```js
function SuperType(name) {
  this.name = name;
  this.colors = ['red', 'blue', 'green'];
}
SuperType.prototype.sayName = function () {
  console.log(this.name);
};
function SubType(name, age) {
  SuperType.call(this, name);
  this.age = age;
}
SubType.prototype = Object.create(SuperType.prototype, { constructor: { value: SubType } })
SubType.prototype.sayAge = function () {
  console.log(this.age);
};
```

这个例子的高效率体现在它只调用了一次 `SuperType` 构造函数，并且因此避免了在 `SubType.prototype` 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 `instanceof` 和 `isPrototypeOf()` 。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。

ECMAScript支持面向对象（OO）编程，但不使用类或者接口。对象可以在代码执行过程中创建和增强，因此具有动态性而非严格定义的实体。在没有类的情况下，可以采用下列模式创建对象。

1. 工厂模式，使用简单的函数创建对象，为对象添加属性和方法，然后返回对象。这个模式后来被构造函数模式所取代。
2. 构造函数模式，可以创建自定义引用类型，可以像创建内置对象实例一样使用 new 操作符。不过，构造函数模式也有缺点，即它的每个成员都无法得到复用，包括函数。由于函数可以不局限于任何对象（即与对象具有松散耦合的特点），因此没有理由不在多个对象间共享函数。
3. 原型模式，使用构造函数的 prototype 属性来指定那些应该共享的属性和方法。组合使用构造函数模式和原型模式时，使用构造函数定义实例属性，而使用原型定义共享的属性和方法。

JavaScript主要通过原型链实现继承。原型链的构建是通过将一个类型的实例赋值给另一个构造函数的原型实现的。这样，子类型就能够访问超类型的所有属性和方法，这一点与基于类的继承很相似。原型链的问题是对象实例共享所有继承的属性和方法因此不适宜单独使用。解决这个问题的技术是借用构造函数，即在子类型构造函数的内部调用超类型构造函数。这样就可以做到个实例都具有自己的属性，同时还能保证只使用构造函数模式来定义类型。使用最多的继承模式是组合继承，这种模式使用原型链继承共享的属性和方法，而通过借用构造函数继承实例属性。

此外，还存在下列可供选择的继承模式。

1. 原型式继承，可以在不必预先定义构造函数的情况下实现继承，其本质是执行对给定对象的浅复制。而复制得到的副本还可以得到进一步改造。
2. 寄生式继承，与原型式继承非常相似，也是基于某个对象或某些信息创建一个对象，然后增强对象，最后返回对象。为了解决组合继承模式由于多次调用超类型构造函数而导致的低效率问题，可以将这个模式与组合继承一起使用。
3. 寄生组合式继承，集寄生式继承和组合继承的优点与一身，是实现基于类型继承的最有效方式。
