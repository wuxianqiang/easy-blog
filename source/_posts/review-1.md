---
title: 高阶前端面试题
date: 2019-12-08 16:33:12
tags: 面试题
---
{% asset_img banner.png 图片 %}

<!-- more -->

## 第一部分

第一题
```js
setTimeout(() => {
  console.log(1)
});
new Promise(resolve => {
  resolve()
  console.log(2)
}).then(() => {
  console.log(3)
  Promise.resolve().then(() => {
    console.log(4)
  }).then(() => {
    Promise.resolve().then(() => {
      console.log(5)
    })
  })
})
```
第二题
```js
var obj = {
  '2': 3,
  '3': 4,
  'length': 2,
  'splice': Array.prototype.splice,
  'push': Array.prototype.push
}
obj.push(1)
obj.push(2)
console.log(obj)
```
第三题
```js
var arr = [[1,2,2],[3,4,5,5],[6,7,8,9,[11,12,[12,13,[14]]]],10]
```
编写一个程序将数组扁平化去除其中重复部分数据，最后得到一个升序且不重复的数组

第四题
改造下面的代码，使之输出0-9，写出你能想到的所有方法
```js
for (var i = 0; i < 10; i++) {
  setTimeout(() => {
    console.log(i)
  }, 1000);
}
```
第五题
```js
var a = 10;
(function () {
  console.log(a);
  a = 5;
  console.log(window.a);
  var a = 20;
  console.log(a)
})()
```
第六题
```js
var a = {},b='123',c=123;
a[b]='b';
a[c]='c';
console.log(a[b])
var a = {},b=Symbol('123'),c=Symbol('123');
a[b]='b';
a[c]='c';
console.log(a[b])
var a = {},b={key: '123'},c={key: '456'};
a[b]='b';
a[c]='c';
console.log(a[b])
```
第七题
```js
function changeObjProperty (o) {
  o.siteUrl='www.baidu.com';
  o = new Object();
  o.siteUrl='www.google.com'
}
let webSite = new Object();
changeObjProperty(webSite);
console.log(webSite.siteUrl)
```
第八题
实现如下继承函数，传入A和B两个类，要求A要继承于B
```js
function inherit (A, B) {

}
```
第九题
实现一个动画函数，实现A元素围绕B元素匀速旋转动画？无需考虑透明度的变化。
```js
function animation(el, start, end, during) {

}
```
第十题
实现一个简易的前端模板引擎
```js
function tpl(template, data) {

}
// 使用
tpl('<div class="{%className%}">{%name%}</div>', {name: 123, className: 'hd'})
// <div class="hd">123</div>
```
## 第二部分

1.有这样一张图片，在只能增加样式的情况下，如何使图片的宽度变成300px呢?
```js
<img src="1.jpg" style="width: 480px!important;">
```
2.`['1', '2', '3'].map(parseInt)`答案是?

3.如何定义 a 变量，才能使 `a==1&&a==2&&a==3` 为 `true`，`a=`？

4.有哪些操作会造成内存泄漏?

5.请描述一下cookie，localStorage，sessionStorage?

6.请指出`window.onload`和document ready的区别?

7.JS延迟加载的方式有哪些?

8.去掉 abc 字符串中的所有空格的计算表达式?

9.RESTful API 接口中常用的几种 HTTP 请求方式分别为?

10.写一个箭头函数实现把一个字符串的所有字母大小写取反?

11.判断一个字符串是不是邮件地址的表达式为?

12.常用的AMD模式的前端脚本库是什么，常用的CMD模式的前端脚本库是什么?

13.闭包是什么?

14.移动端页面常用什么来实现页面自适应?

15.代码的执行结果是什么?
```js
var myObject = {
  foo: 'bar',
  func: function () {
    var self = this;
    console.log('outer func: this.foo = ' + this.foo);
    console.log('outer func: self.foo = ' + self.foo);
    (function (){
      console.log('outer func: this.foo = ' + this.foo);
      console.log('outer func: self.foo = ' + self.foo);
    }())
  }
}
myObject.func()
```
## 第三部分

CSS3的弹性盒子布局中内容对齐属性justify-content里能平均分布一行，而且首尾留有间隙的属性值是：

A：center
B：space-around
C：space-between
D：flex-end



有个HTML标签其所在页面定义的样式：
```js
<style>
    a {color: yellow;}
    #b {color: green;}
    c {color: blue;}
</style>

<a id="b" class="c" style="color: red"></a>
```
请问最终呈现时a标签的文字颜色为

A：yellow
B：green
C：blue
D：red



代码的最后执行结果是多少
```js
var a = {name: '前端开发'};
var b = a;
a = null;
console.log(b)
```
A：null
B：{name: '前端开发'}
C：undefined
D：a



Promise的构造函数是（），那么then方法是（）
A：同步执行
B：异步执行
C：不确定



`a.b.c.d`和`a[b][c][d]`哪个性能更高
A：`a.b.c.d`
B：`a[b][c][d]`
C：性能一样



代码的最后执行结果是多少
```js
function changeObjProperty(o) {
  o.siteUrl = 'www.baidu.com';
  o = new Object();
  o.siteUrl = 'www.google.com'
}

let webSite = new Object();
changeObjProperty(webSite);
console.log(webSite.siteUrl);
```
A：www.baidu.com
B：www.google.com
C：undefined



display: none;的特点是（）
visibility: hidden;的特点是（）
opacity: 0;的特点是（）

A：不占空间，不能点击
B：占空间，不能点击
C：不占空间，能点击
D：占空间，能点击



NodeJS适用于哪些应用场景

A：高并发
B：多任务
C：I/O 吞吐密集
D：高稳定性
