---
title: 给你一份大厂的面试题
date: 2019-11-03 09:30:52
tags: 面试题
---
{% asset_img banner.png 图片 %}
<!-- more -->

第一题：写出下面输出的内容
```js
console.log('begin')
for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i);
  }, 1000)
}
console.log('end')
```


第二题：点击li输出里面的文字
```js
<ul id="container">
  <li>这是第一条</li>
  <li>这是第二条</li>
  <li>这是第三条</li>
</ul>
```


第三题：传入A和B两个类，实现A继承B
```js
function inherit(A, B) {
  // TODO
}
```


第四题：实现宽高100px的弹窗在屏幕上下左右居中
```css
.layer {
  width: 100px;
  height: 100px;
  // TODO
}
```


第五题：实现一个动画函数，A元素围绕B元素匀速运动。
```js
function animation(el, start, end, during) {
  // TODO
}
```


第六题：实现一个前端模板引擎
```js
function tpl(template, data) {
  // TODO
}

tpl(
  '<div class="{%className%}">{%name%}</div>',
  {name: 123, className: 'hd'}
)
```


第七题：写出最后输出结果
```js
var a = 1;
function Fn1() {
  var a = 2;
  console.log(this.a + a);
}
function Fn2() {
  var a = 10;
  Fn1()
}
Fn2()
var Fn3 = function () {
  this.a = 3
}
Fn3.prototype = {
  a: 4
}
var fn3 = new Fn3();
Fn1.call(fn3)
```


第八题：实现concat函数，将两个数组合并
```js
const A = [1, 3, 5, 7, 8, 9, 15]
const B = [2, 4, 6, 8, 13]
function concat(A, B) {
  // TODO
}
```


第九题：实现reverse函数，将二叉树左右翻转
```js
var node1 = {
  value: 1,
  left: {
    value: 2,
    left: {
      value: 4
    },
    right: {
      value: 5
    }
  },
  right: {
    value: 3,
    left: {
      value: 6
    },
    right: {
      value: 7
    }
  }
}

function reverse (node) {
  // TODO
}
```
