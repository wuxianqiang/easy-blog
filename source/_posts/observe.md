---
title: 设计模式之观察者模式
date: 2019-09-01 12:20:20
tags:
---
{% asset_img banner.png 图片 %}

观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状 态发生改变时，所有依赖于它的对象都将得到通知。

<!-- more -->

浏览器采用了类似的机制。事件可能来自用户的 点击或者加载某些文件时产生，而这些产生的事 件都有对应的观察者。

在Node中，事件主要来源于网络请求、文件 I/O 等，这些事件对应的观察者 有文件 I/O 观察者、网络 I/O 观察者等。观察者将事件进行了分类。

```js
document.body.addEventListener( 'click', function(){
  alert(2);
}, false );
```

在这里需要监控用户点击 `document.body` 的动作，但是我们没办法预知用户将在什么时候点击。所以我们订阅 `document.body` 上的 `click` 事件，当 `body` 节点被点击时， `body` 节点便会向订阅者发布这个消息。

假如我们正在开发一个商城网站，网站里有 header 头部、nav 导航、消息列表、购物车等模块。这几个模块的渲染有一个共同的前提条件，就是必须先用 ajax 异步请求获取用户的登录信息。这是很正常的，比如用户的名字和头像要显示在 header模块里，而这两个字段都来自用户登录后返回的信息。

如果它们和用户信息模块产生了强耦合，比如下面这样的形式：

```js
login.succ(function(data){
  header.setAvatar( data.avatar); // 设置 header 模块的头像
  nav.setAvatar( data.avatar ); // 设置导航模块的头像
});
```

至于 ajax 请求什么时候能成功的返回用户信息，这点我们没有办法确定。所以使用事件通知机制去处理（观察者模式）

```js
function Observer(){
  this.state = null;
  this.arr = [];
}
Observer.prototype.attach = function(s){
  this.arr.push(s);
}
Observer.prototype.setState = function(newState){
  this.state = newState;
  this.arr.forEach(s=>s.update(this.state));
}

function Subject(name){
  this.name = name;
}

Subject.prototype.setAvatar = function (data) {
  console.log(`设置 ${this.name} 模块的头像`)
}

Subject.prototype.update = function(newState){
  this.setAvatar(newState.avatar)
}
let o = new Observer();
let s1 = new Subject('nav');
let s2 = new Subject('header');
o.attach(s1);
o.attach(s2);
o.setState({avatar: '数据'});
```

观察者模式的优点非常明显，一为时间上的解耦，二为对象之间的解耦。它的应用非常 广泛，既可以用在异步编程中，也可以帮助我们完成更松耦合的代码编写。

当然，观察者模式也不是完全没有缺点。创建订阅者本身要消耗一定的时间和内存，而且当你订阅一个消息后，也许此消息最后都未发生，但这个订阅者会始终存在于内存中。

另外，观察者模式虽然可以弱化对象之间的联系，但如果过度使用的话，对象和对象之间的必要联系也将被深埋在背后，会导致程序难以跟踪维护和理解。特别是有多个发布者和订阅者嵌套到一起的时候，要跟踪一个 bug不是件轻松的事情。
