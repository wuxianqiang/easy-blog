---
title: 执行环境，作用域链，闭包
date: 2019-08-17 18:52:47
tags: JavaScript
---
{% asset_img banner.jpg 图片 %}

<!-- more -->

## 执行环境

执行环境是JavaScript中最为重要的一个概念。执行环境定义了变量或函数有权访问的其他数据，决定了它们各自的行为。每个执行环境都有一个与之关联的变量对象，环境中定义的所有变量和函数都保存在这个对象中。虽然我们编写的代码无法访问这个对象，但解析器在处理数据时会在后台使用它。

某个执行环境中的所有代码执行完毕后，该环境被销毁，保存在其中的所有变量和函数定义也随之销毁。

每个函数都有自己的执行环境。当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。而在函数执行之后，栈将其环境弹出，把控制权返回给之前的执行环境。

如果这个环境是函数，则将其活动对象作为变量对象。活动对象在最开始时只包含一个变量，即 arguments 对象（这个对象在全局环境中是不存在的）。作用域链中的下一个变量对象来自包含（外部）环境，而再下一个变量对象则来自下一个包含环境。这样，一直延续到全局执行环境；全局执行环境的变量对象始终都是作用域链中的最后一个对象。

## 作用域链

```js
function compare(value1, value2){
    if (value1 < value2){
        return -1;
    } else if (value1 > value2){
        return 1;
    } else {
        return 0;
    }
}
var result = compare(5, 10);
```

以上代码先定义了 `compare()` 函数，然后又在全局作用域中调用了它。当第一次调用 `compare()` 时，会创建一个包含 `this` 、 `arguments` 、 `value1` 和 `value2` 的活动对象。全局执行环境的变量对象（包含 `this` 、 `result` 和 `compare` ）在 `compare()` 执行环境的作用域链中则处于第二位。下图展示了包含上述关系的 `compare()` 函数执行时的作用域链。

{% asset_img scope1.png 图片 %}

对于这个例子中 `compare()` 函数的执行环境而言，其作用域链中包含两个变量对象：本地活动对象和全局变量对象。显然，作用域链本质上是一个指向变量对象的指针列表，它只引用但不实际包含变量对象。

无论什么时候在函数中访问一个变量时，就会从作用域链中搜索具有相应名字的变量。一般来讲，当函数执行完毕后，局部活动对象就会被销毁，内存中仅保存全局作用域（全局执行环境的变量对象）。但是，闭包的情况又有所不同。

## 闭包

```js
function createComparisonFunction(propertyName) {
    return function(object1, object2){
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];
        if (value1 < value2){
            return -1;
        } else if (value1 > value2){
            return 1;
        } else {
            return 0;
        }
    };
}
```

闭包是指有权访问另一个函数作用域中的变量的函数。创建闭包的常见方式，就是在一个函数内部创建另一个函数。

在另一个函数内部定义的函数会将包含函数（即外部函数）的活动对象添加到它的作用域链中。

因此，在 `createComparisonFunction()` 函数内部定义的匿名函数的作用域链中，实际上将会包含外部函数 `createComparisonFunction()` 的活动对象。下图展示了当下列代码执行时，包含函数与内部匿名函数的作用域链。

```js
var compare = createComparisonFunction(“name”);
var result = compare({ name: “Nicholas” }, { name: “Greg” });
```

{% asset_img scope2.png 图片 %}


在匿名函数从 `createComparisonFunction()` 中被返回后，它的作用域链被初始化为包含 `createComparisonFunction()` 函数的活动对象和全局变量对象。这样，匿名函数就可以访问在 `createComparisonFunction()` 中定义的所有变量。更为重要的是， `createComparisonFunction()` 函数在执行完毕后，其活动对象也不会被销毁，因为匿名函数的作用域链仍然在引用这个活动对象。换句话说，当 `createComparisonFunction()` 函数返回后，其执行环境的作用域链会被销毁，但它的活动对象仍然会留在内存中；直到匿名函数被销毁后， `createComparisonFunction()` 的活动对象才会被销毁

由于闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存。过度使用闭包可能会导致内存占用过多，我们建议读者只在绝对必要时再考虑使用闭包。虽然像V8等优化后的JavaScript引擎会尝试回收被闭包占用的内存，但请大家还是要慎重使用闭包。
