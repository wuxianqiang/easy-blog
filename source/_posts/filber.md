---
title: 时间分片
date: 2020-03-14 12:17:02
tags: JavaScript
---
{% asset_img banner.png 图片 %}

在[《浏览器UI线程更新机制》]([https://mp.weixin.qq.com/s/GdUJOu6aMQEmi05jQf8Sbw)文章中介绍大多数浏览器在 JavaScript 运行时停止 UI 线程队列中的任务，也就是说 JavaScript 任务必须尽快结束，以免对用户体验造成不良影响。

<!-- more -->

尽管你尽了最大努力，还是有一些 JavaScript 任务因为复杂性原因不能在 100 毫秒或更少时间内完成。这种情况下，理想方法是让出对 UI 线程的控制，使 UI 更新可以进行。让出控制意味着停止 JavaScript 运行，给 UI 线程机会进行更新，然后再继续运行 JavaScript。于是 JavaScript 定时器进入了我们的视野。



定时器与 UI 线程交互的方式有助于分解长运行脚本成为较短的片断。调用 setTimeout() 或 setInterval() 告诉 JavaScript 引擎等待一定时间然后将 JavaScript 任务添加到 UI 队列中。例如：


```js
function greeting() {
  alert("Hello world!");
}
setTimeout(greeting, 250);
```

此代码将在 250 毫秒之后，向 UI 队列插入一个 JavaScript 任务运行 greeting() 函数。在那个点之前，所有其他 UI 更新和 JavaScript 任务都在运行。请记住，第二个参数指什么时候应当将任务添加到 UI 队列之中，并不是说那时代码将被执行。这个任务必须等到队列中的其他任务都执行之后才能被执行。



一个常见的长运行脚本就是循环占用了太长的运行时间。那么定时器就是你的下一个优化步骤。其基本方法是将循环工作分解到定时器序列中。典型的循环模式如下：


```js
for (var i = 0, len = items.length; i < len; i++) {
  process(items[i]);
}
```

这样的循环结构运行时间过长的原因有二，process() 的复杂度，items 的大小，或两者兼有。



此处理过程必须是同步处理吗？数据必须按顺序处理吗？如果这两个回答都是“否”，那么代码将适于使用定时器分解工作。一种基本异步代码模式如下：


```js
function processArray(items, process, callback) {
  var todo = items.concat();
  function updateProgress () {
    if (todo.length > 0) {
      process(todo.shift());
      setTimeout(updateProgress, 25);
    } else {
      callback()
    }
  }
  updateProgress()
}
// 用例
processArray([1, 2, 3, 4], (item) => {
  console.log('当前项', item)
}, () => {
  console.log('执行完成')
})
```

在 Windows 系统上定时器分辨率为 15 毫秒，也就是说一个值为 15 的定时器延时将根据最后一次系统时间刷新而转换为 0 或者 15。设置定时器延时小于 15 将在 Internet Explorer 中导致浏览器锁定，所以最小值建议为 25 毫秒（实际时间是 15 或 30）以确保至少 15 毫秒延迟。



我们通常将一个任务分解成一系列子任务。如果一个函数运行时间太长，那么查看它是否可以分解成一系列能够短时间完成的较小的函数。可将一行代码简单地看作一个原子任务，多行代码组合在一起构成一个独立任务。某些函数可基于函数调用进行拆分。



其实上面代码存在一个问题，就是定时器的间隔时间问题，我设置的是25毫秒，也就是认为最快是25毫秒可以完成UI更新，但是如果UI更新比25毫秒还要快，那么就需要等待一段时间，这个等待时间是没有必要的，所以希望的是UI更新完成之后就立马执行。而实现的关键是两个新API。


```
requestIdleCallback 事件循环空闲期的回调函数

requestAnimationFrame 在UI更新完成之后的回调
```

所以根据实际情况去选择API，在我们任务分解的过程中我们可以把定时器改成 requestAnimationFrame，如果考虑兼容问题，那还是使用定时器吧。


```js
function processArray(items, process, callback) {
  var todo = items.concat();
  function updateProgress () {
    if (todo.length > 0) {
      process(todo.shift());
      requestAnimationFrame(updateProgress)
    } else {
      callback()
    }
  }
  updateProgress()
}
```

解决同步阻塞的方法，通常有两种: 异步 与 任务分割。而 React Fiber 便是为了实现任务分割而诞生的。



Fiber 其实可以算是一种编程思想，在其它语言中也有许多应用(Ruby Fiber)。核心思想是任务拆分和协同，主动把执行权交给主线程，使主线程有时间空挡处理其他高优先级任务。当遇到进程阻塞的问题时，任务分割、异步调用和缓存策略是三个显著的解决思路。
