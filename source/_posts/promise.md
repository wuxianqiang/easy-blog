---
title: promise A+ 原理
date: 2019-06-30 21:28:10
tags: JavaScript
---

{% asset_img banner.png 图片 %}

[规范的文档说明](https://segmentfault.com/a/1190000002452115)

1. 第一步实现简单调用，成功调用resolve，失败调用reject，参数传入then的回调函数中。

<!-- more -->

```js
function Promise (executor) {
  let self = this;
  self.status = 'pending';
  self.value = undefined;
  self.reason = undefined;
  function resolve (value) {
    self.value = value
    self.status = 'fulfilled'
  }
  function reject (reason) {
    self.reason = reason
    self.status = 'rejected'
  }
  try {
    executor(resolve, reject)
  } catch (error) {
    reject(error)
  }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
  let self = this;
  if (self.status === 'fulfilled') {
    onFulfilled(self.value)
  }
  if (self.status === 'rejected') {
    onRejected(self.reason)
  }
}
```
测试代码
```js
let Promise = require('./index.js')

let p = new Promise((resolve, reject) => {
  resolve(100)
})

p.then((data) => {
  console.log('成功', data)
}, (err) => {
  console.log('失败', err)
})
```
2. 为了防止resolve和reject被多次调用，需要保证状态唯一
```js
function Promise (executor) {
  let self = this;
  self.status = 'pending';
  self.value = undefined;
  self.reason = undefined;
  function resolve (value) {
    if (self.status === 'pending') {
      self.value = value
      self.status = 'fulfilled'
    }
  }
  function reject (reason) {
    if (self.status === 'pending') {
      self.reason = reason
      self.status = 'rejected'
    }
  }
  try {
    executor(resolve, reject)
  } catch (error) {
    reject(error)
  }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
  let self = this;
  if (self.status === 'fulfilled') {
    onFulfilled(self.value)
  }
  if (self.status === 'rejected') {
    onRejected(self.reason)
  }
}
```
测试代码
```js
let Promise = require('./index.js')

let p = new Promise((resolve, reject) => {
  resolve(100);
  reject(50)
})

p.then((data) => {
  console.log('成功', data)
}, (err) => {
  console.log('失败', err)
})
```
3. 异步处理，then回调函数的参数还没有值，状态是pending，通过发布订阅模式处理回调
```js
function Promise (executor) {
  let self = this;
  self.status = 'pending';
  self.value = undefined;
  self.reason = undefined;
  self.onFulfilledCallbacks = [];
  self.onRejectedCallbacks = [];
  function resolve (value) {
    if (self.status === 'pending') {
      self.value = value
      self.status = 'fulfilled';
      self.onFulfilledCallbacks.forEach(fn => fn())
    }
  }
  function reject (reason) {
    if (self.status === 'pending') {
      self.reason = reason
      self.status = 'rejected'
      self.onRejectedCallbacks.forEach(fn => fn())
    }
  }
  try {
    executor(resolve, reject)
  } catch (error) {
    reject(error)
  }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
  let self = this;
  if (self.status === 'fulfilled') {
    onFulfilled(self.value)
  }
  if (self.status === 'rejected') {
    onRejected(self.reason)
  }
  if (self.status === 'pending') {
    self.onFulfilledCallbacks.push(function () {
      onFulfilled(self.value)
    });
    self.onRejectedCallbacks.push(function () {
      onRejected(self.reason)
    })
  }
}
```
测试代码
```js
let Promise = require('./index.js')

let p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(100);
  }, 2000);
})

p.then((data) => {
  console.log('成功', data)
}, (err) => {
  console.log('失败', err)
})
```
处理then回调的返回值类型，递归解析处理值为普通值
```js
function Promise (executor) {
  let self = this;
  self.status = 'pending';
  self.value = undefined;
  self.reason = undefined;
  self.onFulfilledCallbacks = [];
  self.onRejectedCallbacks = [];
  function resolve (value) {
    if (self.status === 'pending') {
      self.value = value
      self.status = 'fulfilled';
      self.onFulfilledCallbacks.forEach(fn => fn())
    }
  }
  function reject (reason) {
    if (self.status === 'pending') {
      self.reason = reason
      self.status = 'rejected'
      self.onRejectedCallbacks.forEach(fn => fn())
    }
  }
  try {
    executor(resolve, reject)
  } catch (error) {
    reject(error)
  }
}

function resolvePromise (promise2, x, resolve, reject) {
  if (promise2 === x) {
    reject('TypeError')
  } else {
    let isChange = false
    if (typeof x !== null && (typeof x === 'function' || typeof x === 'object')) {
      try {
        // 对象区属性可能会报错
        let then = x.then
        if (typeof then === 'function') {
          then.call(x, function (y) {
            resolvePromise(promise2, y, resolve, reject)
          }, function (r) {
            if (isChange) return
            isChange = true
            reject(r)
          })
        } else {
          if (isChange) return
          isChange = true
          resolve(x)
        }
      } catch (error) {
        if (isChange) return
        isChange = true
        reject(error)
      }
    } else {
      if (isChange) return
      isChange = true
      resolve(x)
    }
  }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
  let self = this;
  let promise2 = new Promise(function (resolve, reject) {
    if (self.status === 'fulfilled') {
      setTimeout(() => {
        try {
          let x = onFulfilled(self.value)
          resolvePromise(promise2, x, resolve, reject)
        } catch (error) {
          reject(error)
        }
      }, 0);
    }
    if (self.status === 'rejected') {
      setTimeout(() => {
        try {
          let x = onRejected(self.reason)
          resolvePromise(promise2, x, resolve, reject)
        } catch (error) {
          reject(error)
        }
      }, 0);
    }
    if (self.status === 'pending') {
      self.onFulfilledCallbacks.push(function () {
        try {
          let x = onFulfilled(self.value)
          resolvePromise(promise2, x, resolve, reject)
        } catch (error) {
          reject(error)
        }
      });
      self.onRejectedCallbacks.push(function () {
        try {
          let x = onRejected(self.reason)
          resolvePromise(promise2, x, resolve, reject)
        } catch (error) {
          reject(error)
        }
      })
    }
  })
  return promise2
}
```
测试代码
```js
let Promise = require('./index.js')

let p = new Promise((resolve, reject) => {
  resolve(100);
})

let promise2 = p.then((data) => {
  return new Promise((resolve, reject) => {
    reject(88)
  })
}).then((data) => {
  console.log(data)
}, (err) => {
  console.log(err)
})
```
处理then回调函数参数的默认值，then里面的回调都是可以不传的
```js
function Promise (executor) {
  let self = this;
  self.status = 'pending';
  self.value = undefined;
  self.reason = undefined;
  self.onFulfilledCallbacks = [];
  self.onRejectedCallbacks = [];
  function resolve (value) {
    if (self.status === 'pending') {
      self.value = value
      self.status = 'fulfilled';
      self.onFulfilledCallbacks.forEach(fn => fn())
    }
  }
  function reject (reason) {
    if (self.status === 'pending') {
      self.reason = reason
      self.status = 'rejected'
      self.onRejectedCallbacks.forEach(fn => fn())
    }
  }
  try {
    executor(resolve, reject)
  } catch (error) {
    reject(error)
  }
}

function resolvePromise (promise2, x, resolve, reject) {
  if (promise2 === x) {
    reject('TypeError')
  } else {
    let isChange = false
    if (typeof x !== null && (typeof x === 'function' || typeof x === 'object')) {
      try {
        // 对象区属性可能会报错
        let then = x.then
        if (typeof then === 'function') {
          then.call(x, function (y) {
            resolvePromise(promise2, y, resolve, reject)
          }, function (r) {
            if (isChange) return
            isChange = true
            reject(r)
          })
        } else {
          if (isChange) return
          isChange = true
          resolve(x)
        }
      } catch (error) {
        if (isChange) return
        isChange = true
        reject(error)
      }
    } else {
      if (isChange) return
      isChange = true
      resolve(x)
    }
  }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : function (value) {
    return value
  }
  onRejected = typeof onRejected === 'function' ? onRejected : function (reason) {
    throw reason
  }
  let self = this;
  let promise2 = new Promise(function (resolve, reject) {
    if (self.status === 'fulfilled') {
      setTimeout(() => {
        try {
          let x = onFulfilled(self.value)
          resolvePromise(promise2, x, resolve, reject)
        } catch (error) {
          reject(error)
        }
      }, 0);
    }
    if (self.status === 'rejected') {
      setTimeout(() => {
        try {
          let x = onRejected(self.reason)
          resolvePromise(promise2, x, resolve, reject)
        } catch (error) {
          reject(error)
        }
      }, 0);
    }
    if (self.status === 'pending') {
      self.onFulfilledCallbacks.push(function () {
        try {
          let x = onFulfilled(self.value)
          resolvePromise(promise2, x, resolve, reject)
        } catch (error) {
          reject(error)
        }
      });
      self.onRejectedCallbacks.push(function () {
        try {
          let x = onRejected(self.reason)
          resolvePromise(promise2, x, resolve, reject)
        } catch (error) {
          reject(error)
        }
      })
    }
  })
  return promise2
}
```
promise类静态方法resolve和reject的原理
```js
Promise.resolve = function () {
  return new Promise(function (resolve, reject) {
    resolve()
  })
}

Promise.reject = function () {
  return new Promise(function (resolve, reject) {
    reject
  })
}
```
promise静态方法all的实现原理是计数器，应为要保证结果的顺序
```js
Promise.all = function (arrFn) {
  return new Promise(function (resolve, reject) {
    let arr = []
    let count = 0
    function handleData (data, index) {
      arr[index] = data
      if (++count === arrFn.length) {
        resolve(arr)
      }
    }
    arrFn.forEach((fn, index) => {
      fn.then((data) => {
        handleData(data, index)
      }, reject)
    })
  })
}
```
promise中cache方法其实就是then方法的简写
```js
Promise.prototype.catch = function (errFn) {
  return this.then(null, errFn)
}
```
finally是promise中无论状态成功还是失败都会执行的
```js
Promise.prototype.finally = function (callback) {
  return this.then((data) => {
    callback()
    return data
  }, (err) => {
    callback()
    return err
  })
}
```
promise中race的调用方式
```js
Promise.race = function (arrFn) {
  return new Promise(function (resolve, reject) {
    arrFn.forEach(fn => {
      fn.then(resolve, reject)
    })
  })
}
```
