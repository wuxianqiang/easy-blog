---
title: this指向
date: 2019-11-30 12:00:15
tags: JavaScript
---
{% asset_img banner.png 图片 %}

类的方法内部如果含有 this，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。

<!-- more -->

```js
class Logger {
 printName(name = 'there') {
   this.print(`Hello ${name}`);
 }

 print(text) {
   console.log(text);
 }
}
const logger = new Logger();
const { printName } = logger;
printName();  // 报错
```


上面代码中，printName 方法中的 this，默认指向 Logger类的实例。但是，如果将这个方法提取出来单独使用，this会指向该方法运行时所在的环境（由于 class 内部是严格模式，所以 this 实际指向的是 undefined），从而导致找不到print 方法而报错。



一个比较简单的解决方法是，在构造方法中绑定 this，这样就不会找不到 print 方法了。
```js
class Logger {
 constructor() {
   this.printName = this.printName.bind(this);
 }
 printName(name = 'there') {
   this.print(`Hello ${name}`);
 }
 print(text) {
   console.log(text);
 }
}
```

通过定义一个实例上的属性去调用原型上的方法。





另一种解决方法是使用箭头函数。此语法需要安装 babel 插件@babel/plugin-proposal-class-properties
```js
class Logger {
  printName = (name = 'there') => {
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}
```


箭头函数内部的 this 总是指向定义时所在的对象。上面代码中，箭头函数位于构造函数内部，它的定义生效的时候，是在构造函数执行的时候。这时，箭头函数所在的运行环境，肯定是实例对象，所以 this 会总是指向实例对象。



还有一种解决方法是使用 Proxy，获取方法的时候，自动绑定 this。
```js
function selfish(target) {
  const cache = new WeakMap();
  const handler = {
    get(target, key) {
      const value = Reflect.get(target, key);
      if (typeof value !== 'function') {
        return value;
      }
      if (!cache.has(value)) {
        cache.set(value, value.bind(target));
      }
      return cache.get(value);
    }
  };
  const proxy = new Proxy(target, handler);
  return proxy;
}

const logger = selfish(new Logger());
let { printName } = logger
printName() // Hello there
```


Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。



上面示例中当取值是函数的时候，将会为函数通过bind方法绑定this。而且使用WeakMap做了优化，通过缓存到这个对象中，下次直接取值，不用每次都绑定，键名是源函数，键值是绑定之后的函数。



首先，WeakMap 只接受对象作为键名（null 除外），不接受其他类型的值作为键名。其次，WeakMap 的键名所指向的对象，不计入垃圾回收机制。WeakMap 结构有助于防止内存泄漏。
