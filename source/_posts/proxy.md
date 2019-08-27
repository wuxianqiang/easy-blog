---
title: 设计模式之代理模式
date: 2019-08-27 22:42:40
tags: JavaScript
---
{% asset_img banner.png 图片 %}

代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问。

<!-- more -->

比如我们想要访问Google服务器，但是在中国不能直接访问，需要配置一个单例服务器访问。

```js
class Google {
  get() {
    return 'google'
  }
}

// 通过代理访问google
class Proxy {
  constructor () {
    this.google = new Google();
  }
  get(url) {
    return this.google.get(url);
  }
}

let proxy = new Proxy();
let result = proxy.get('www.google.com');
console.log(result) // google
```

事件委托也是代理模式的体现，比如里层元素绑定事件的时候可以代理到外层元素。

```js
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</ul>

<script>
  let list = document.getElementById('list');
  list.addEventListener('click', function (event) {
    alert(event.target.innerHTML)
  })
</script>
```
通过这个代理对象，在图片被真正加载好之前，页面中
将出现一张占位的loading, 来提示用户图片正在加载。代码如下：
```js
var myImage = (function () {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  return {
    setSrc: function (src) {
      imgNode.src = src;
    }
  }
})();
var proxyImage = (function () {
  var img = new Image;
  img.onload = function () {
    myImage.setSrc(this.src);
  }
  return {
    setSrc: function (src) {
      myImage.setSrc('./loading.png');
      img.src = src;
    }
  }
})();
proxyImage.setSrc('http://banner.png');
```
在使用Vue时候data里面的的数据都会代理到Vue实例中，就是为了方便用户获取。
```js
let vm = new Vue({
  data: {
    msg: 'hello'
  }
})
console.log(vm.$data.msg)
console.log(vm.msg)
```
在使用Koa的时候content上下文中会代理封装在请求request和响应response里面的属性。
```js
app.use((ctx) => {
  console.log(ctx.request.url)
  console.log(ctx.url)
  console.log(ctx.response.body)
  console.log(ctx.body)
})
```
代理模式是一种非常有意义的模式，在框架中可以找到很多代理模式的场景。
