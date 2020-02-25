---
title: Promise的10大知识点！
date: 2020-02-25 09:38:07
tags: JavaScript
---
{% asset_img banner.png 图片 %}
<!-- more -->
### Part 1:
```js
const prom = new Promise((res, rej) => {
  console.log('first');
  res();
  console.log('second');
});
prom.then(() => {
  console.log('third');
});
console.log('fourth');

// first
// second
// fourth
// third
```
> 知识点：Promise 构造函数是同步执行，promise.then  是异步执行。
### Part 2:
```js
const prom = new Promise((res, rej) => {
  setTimeout(() => {
    res('success');
  }, 1000);
});
const prom2 = prom.then(() => {
  throw new Error('error');
});

console.log('prom', prom);
console.log('prom2', prom2);

setTimeout(() => {
  console.log('prom', prom);
  console.log('prom2', prom2);
}, 2000);

// prom Promise {<pending>}
// prom2 Promise {<pending>}
// Promise {<resolved>: "success"}
// prom2 Promise {<rejected>: Error: error }
```
> 知识点：promise 有三种不同的状态：pending、fulfilled、rejected，一旦状态更新，就不再改变，只能从pending->fulfilled 或 pending->rejected 。
### Part 3:
```
const prom = new Promise((res, rej) => {
  res('1');
  rej('error');
  res('2');
});

prom
  .then(res => {
    console.log('then: ', res);
  })
  .catch(err => {
    console.log('catch: ', err);
  });

// then: 1
```
> 知识点：一旦 resolve 和 reject 同时存在，只会执行第一个，后面的不再执行。
### Part 4:
```js
Promise.resolve(1)
  .then(res => {
    console.log(res);
    return 2;
  })
  .catch(err => {
    return 3;
  })
  .then(res => {
    console.log(res);
  });

// 1
// 2
```
> 知识点：每次 promise 调用 `.then` 或 `catch` 时，都会返回一个新的 promise，从而实现链接调用。
### Part 4:
```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('first')
    resolve('second')
  }, 1000)
})

const start = Date.now()
promise.then((res) => {
  console.log(res, Date.now() - start, "third")
})
promise.then((res) => {
  console.log(res, Date.now() - start, "fourth")
})

// first
// second 1054 third
// second 1054 fourth
```
> 知识点：一个 promise `.then` 或 `.catch` 可以被多次调用，但是此处Promise构造函数仅执行一次。 换句话说，一旦promise 的内部状态发生变化并获得了一个值，则随后对 `.then` 或 `.catch` 的每次调用都将直接获取该值。
### Part 6:
```js
const promise = Promise.resolve()
  .then(() => {
    return promise
  })
promise.catch(console.error)

// [TypeError: Chaining cycle detected for promise #<Promise>]
```

> 知识点：`.then` 或 `.catch` 返回的值不能是 promise 本身，否则将导致无限循环。
### Part 7:
```js
Promise.resolve()
  .then(() => {
    return new Error('error');
  })
  .then(res => {
    console.log('then: ', res);
  })
  .catch(err => {
    console.log('catch: ', err);
  });

// then: Error: error!
```

> 知识点：在 `.then` 或 `.catch` 中返回错误对象不会引发错误，因此后续的 `.catch` 不会捕获该错误对象，您需要更改为以下对象之一：

```js
return Promise.reject(new Error('error'))
throw new Error('error')
```
因为返回任何非promise值都将被包装到一个Promise对象中，也就是说，返回 `new Error('error')` 等同于返回`Promise.resolve(new Error('error'))`。
### Part 8:
```js
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)

  // 1
```
> 知识点：`.then` 或 `.catch` 的参数应为函数，而传递非函数将导致值的结果被忽略，例如 `.then(2)`或 `.then(Promise.resolve(3))`。
### Part 9:
```js
Promise.resolve()
  .then(
    function success(res) {
      throw new Error('Error after success');
    },
    function fail1(e) {
      console.error('fail1: ', e);
    }
  )
  .catch(function fail2(e) {
    console.error('fail2: ', e);
  });

//   fail2:  Error: Error after success
```
> 知识点：`.then` 可以接受两个参数，第一个是处理成功的函数，第二个是处理错误的函数。 `.catch` 是编写 `.then` 的第二个参数的便捷方法，但是在使用中要注意一点：`.then` 第二个错误处理函数无法捕获第一个成功函数和后续函数抛出的错误。 `catch` 捕获先前的错误。 当然，如果要重写，下面的代码将起作用：

```js
Promise.resolve()
  .then(function success1 (res) {
    throw new Error('success1 error')
  }, function fail1 (e) {
    console.error('fail1: ', e)
  })
  .then(function success2 (res) {
  }, function fail2 (e) {
    console.error('fail2: ', e)
  })
```
### Part 10:
```js
process.nextTick(() => {
  console.log('1')
})
Promise.resolve()
  .then(() => {
    console.log('2')
  })
setImmediate(() => {
  console.log('3')
})
console.log('4');

// 4 1 2 3
```
> 知识点：`process.nextTick` 和 `promise.then` 都属于微任务，而 `setImmediate` 属于宏任务，它在事件循环的检查阶段执行。 在事件循环的每个阶段（宏任务）之间执行微任务，并且事件循环的开始执行一次。


博主发布的文章同时在手机微信公众同时发布，想了解博主更多前端的知识讲解，欢迎扫码屏幕下方的二维码关注！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200224222231299.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1X3hpYW5xaWFuZw==,size_16,color_FFFFFF,t_70)
