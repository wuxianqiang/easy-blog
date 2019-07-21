---
title: 偏函数的用法
date: 2019-07-21 17:31:22
tags: JavaScript
---
{% asset_img banner.png 图片 %}

偏函数用法是指创建一个调用另外一个部分——参数或变量已经预置的函数——的函数的用法。

<!-- more -->

```js
let toString = Object.prototype.toString;
let isString = function (obj) {
  return toString.call(obj) == '[object String]';
};
let isFunction = function (obj) {
  return toString.call(obj) == '[object Function]';
};
```

在JavaScript中进行类型判断时，我们通常会进行类似上述代码的方法定义。这段代码固然不复杂，只有两个函数的定义，但是里面存在的问题是我们需要重复去定义一些相似的函数，如果有更多的 isXXX() ，就会出现更多的冗余代码。

为了解决重复定义的问题，我们引入一个新函数，这个新函数可以如工厂一样批量创建一些类似的函数。在下面的代码中，我们通过 `isType()` 函数预先指定 `type` 的值，然后返回一个新的函数：

```js
let isType = function (type) {
  return function (obj) {
    return toString.call(obj) == '[object ' + type + ']';
  };
};

let isString = isType('String');
let isFunction = isType('Function');
```

可以看出，引入 `isType()` 函数后，创建 `isString()` 、 `isFunction()` 函数就变得简单多了。这种通过指定部分参数来产生一个新的定制函数的形式就是偏函数。

偏函数应用在异步编程中也十分常见，著名类库 lodash 提供的 `after()` 方法即是偏函数应用，其定义如下：


```js
let after = function (times, task) {
  return function () {
    if (times-- === 1) {
      return task.call(this, arguments)
    }
  }
}
```

这个函数可以根据传入的 times 参数和具体方法，生成一个需要调用多次才真正执行实际函数的函数。
