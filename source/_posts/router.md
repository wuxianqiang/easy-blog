---
title: 轻松理解路由原理
date: 2019-08-01 08:12:17
tags: JavaScript
---

{% asset_img banner.jpg 图片 %}

路由有hash路由和history路由两种模式。

<!-- more -->

## hash路由

简单模拟路由实现原理。
```js
<body>
  <a href="#/home">首页</a>
  <a href="#/about">关于</a>
  <div id="html"></div>

  <script>
    window.addEventListener('load', () => {
      html.innerHTML = location.hash.slice(1)
    })
    window.addEventListener('hashchange', () => {
      html.innerHTML = location.hash.slice(1)
    })
</script>
</body>
```
通过 `hashchange` 事件，可以知道 URL 的参数什么时候发生了变化，即什么时候该有所反应。

## history路由
简单模拟路由实现原理。

```js
<body>
  <a onclick="go('/home')">首页</a>
  <a onclick="go('/about')">关于</a>
  <div id="html"></div>

  <script>
    function go(pathname) {
      html.innerHTML = pathname
      history.pushState({}, null, pathname)
    }
    window.addEventListener('load', () => {
      go(location.pathname)
    })
    window.addEventListener('popstate', () => {
      go(location.pathname)
    })
</script>
</body>
```

HTML5而通过状态管理API，能够在不加载新页面的情况下改变浏览器的URL。为此，需要使用 `history.pushState()` 方法，该方法可以接收三个参数：状态对象、新状态的标题和可选的相对URL。例如：

```js
history.pushState({state: 'aa'}, '标题',  '/home');
```

执行 `pushState()` 方法后，新的状态信息就会被加入历史状态栈，而浏览器地址栏也会变成新的相对URL。但是，浏览器并不会真的向服务器发送请求，即使状态改变 之后查询 `location.href` 也会返回与地址栏中相同的地址。另外，第二个参数目前还没有浏览器实现，因此完全可以只传入一个空字符串，或者一个短标题也可以。而 第一个参数则应该尽可能提供初始化页面状态所需的各种信息。

因为 `pushState()` 会创建新的历史状态，所以你会发现“后退”按钮也能使用了。按下“后退”按钮，会触发 `window` 对象的 `popstate` 事件。 `popstate` 事件的事件对象有一个 state 属性，这个属性就包含着当初以第一个参数传递给 `pushState()` 的状态对象。

要更新当前状态，可以调用 `replaceState()` ，传入的参数与 `pushState()` 的前两个参数相同。调用这个方法不会在历史状态栈中创建新状态，只会重写当前状态。

在使用HTML5的状态管理机制时，请确保使用 `pushState()` 创造的每一个“假”URL，在Web服务器上都有一个真的、实际存在的URL与之对应。否 则，单击“刷新”按钮会导致404错误。
