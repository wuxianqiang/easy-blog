---
title: Node中的事件循环
date: 2019-06-30 21:02:27
tags: Node
---

{% asset_img banner.png 图片 %}

Node的自身执行模型是事件循环，理解了事件循环可以清楚的知道代码的执行顺序。事件循环就像一个这样的循环体，不断的轮询。当然，如果没有观察者，进程就会退出，不会死循环的。哈哈。

<!-- more -->

```js
while (true) {
  // 执行异步操作
}
```
事件循环里面都有对应的观察者，然后事件循环从观察者中取出事件并执行。观察者可以理解为一个数据存在一对多的关系，所以使用了观察者。观察者是有先后时序的。

{% asset_img loop.png 图片 %}

1. `timers` 观察者中存放了定时器（setTimeout，setInterval）回调队列
2. `idle` 观察者中存放 `process.nextTick()` 回调队列
2. `poll` 观察者中存放了读取文件的回调队列
3. `check` 观察者中存放 `setImmediate` 回调队列

现在已经知道了观察者执行的先后顺序，下面看看代码演示

```js
// 加入两个nextTick()的回调函数
process.nextTick(function () {
  console.log('nextTick延迟执行1');
});
process.nextTick(function () {
  console.log('nextTick延迟执行2');
});
// 加入两个setImmediate()的回调函数
setImmediate(function () {
  console.log('setImmediate延迟执行1');
  // 进入下次循环
  process.nextTick(function () {
    console.log('最后执行');
  });
});
setImmediate(function () {
  console.log('setImmediate延迟执行2');
});
console.log('正常执行');
```

输出结果

```js
正常执行
nextTick延迟执行1
nextTick延迟执行2
setImmediate延迟执行1
setImmediate延迟执行2
最后执行
```

提示：观察者的回调队列是一个队列执行完毕再执行下一个回调队列。代码中`idle`观察者中有两个`process.nextTick`的回调，`check`观察者中有两个`setImmediate`的回调。如果在一个方法中有回调加入其他队列，但是当前队列有方法没有执行完毕，需要先把当前的执行完毕后再执行其他队列的方法。

总结：Node中包含一些异步的API，而处理异步是通过事件循环的方式，异步API中的回调会存放在对应的观察者队列中，观察者有先后顺序，然后事件循环从观察者中取出事件并执行。
