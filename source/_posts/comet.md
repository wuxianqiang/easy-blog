---
title: 双向通信
date: 2019-07-03 09:52:13
tags: HTTP
---
{% asset_img banner.jpg 图片 %}

WebSocket 与传统 HTTP 有如下好处。

1. 客户端与服务器端只建立一个 TCP 连接，可以使用更少的连接。
2. WebSocket 服务器端可以推送数据到客户端，这远比 HTTP 请求响应模式更灵活、更高效。
3. 有更轻量级的协议头，减少数据传送量。

<!-- more -->

WebSocket 最早是作为 HTML5 重要特性而出现的，现代浏览器大多都支持 WebSocket 协议，接下来我们用一段代码来展现 WebSocket 在客户端的应用示例：

```js
let ws = new WebSocket('ws://localhost:8888')
ws.onopen = function () {
  console.log('客户端链接成功')
  ws.send('hello')
}
ws.onmessage = function (event) {
  console.log('客户端收到服务端的消息', event.data)
}
```

上述代码中，浏览器与服务器端创建 WebSocket 协议请求，在请求完成后连接打开，向服务器端发送一次数据，同时可以通过 `onmessage()` 方法接收服务器端传来的数据。这行为与 TCP 客户端十分相似，相较于 HTTP，它能够双向通信。浏览器一旦能够使用 WebSocket，可以想象应用的使用空间极大。


在 WebSocket 之前，网页客户端与服务器端进行通信最高效的是 Comet 技术。Comet是一种用于web的推送技术，能使服务器能实时地将更新的信息传送到客户端，而无须客户端发出请求，目前有三种实现方式：轮询（polling） 长轮询（long-polling）和iframe流（streaming）。

## 轮询

轮询是客户端和服务器之间会一直进行连接，每隔一段时间就询问一次。

这种方式连接数会很多，一个接受，一个发送。而且每次发送请求都会有 Http 的 Header，会很耗流量，也会消耗 CPU 的利用率。

```js
let express = require('express')
const app = express()
const path = require('path')

// 每次请求都立刻返回
app.get('/clock', function (req, res) {
  res.send(new Date().toLocaleString())
})

app.use(express.static(path.resolve(__dirname, 'public')))
app.listen(8080)
```
```js
<body>
  <div id="app"></div>
  <script>
    // 每1秒向服务器请求时间
    setInterval(() => {
      let xhr = new XMLHttpRequest()
      xhr.open('GET', '/clock', true)
      xhr.onreadystatechange = function () {
        if (xhr.readyState === 4 && xhr.status === 200) {
          app.innerHTML = xhr.responseText
        }
      }
      xhr.send(null)
    }, 1000)
  </script>
</body>
```

## 长轮询

长轮询是对轮询的改进版，客户端发送HTTP给服务器之后，看有没有新消息，如果没有新消息，就一直等待。

当有新消息的时候，才会返回给客户端。在某种程度上减小了网络带宽和CPU利用率等问题。

由于 http 数据包的头部数据量往往很大（通常有400多个字节），但是真正被服务器需要的数据却很少（有时只有 10 个字节左右），这样的数据包在网络上周期性的传输，难免对网络带宽是一种浪费

```js
let express = require('express')
const app = express()
const path = require('path')

// 1秒之后更新数据才返回
app.get('/clock', function (req, res) {
    setTimeout(() => {
      res.send(new Date().toLocaleString())
    }, 1000);
})

app.use(express.static(path.resolve(__dirname, 'public')))
app.listen(8080)
```
```js
<body>
  <div id="app"></div>
  <script>
	// 请求完成之后才发送下次请求
    (function poll () {
      let xhr = new XMLHttpRequest()
      xhr.open('GET', '/clock', true)
      xhr.onreadystatechange = function () {
        if (xhr.readyState === 4 && xhr.status === 200) {
          app.innerHTML = xhr.responseText
          poll()
        }
      }
      xhr.send(null)
    })()
  </script>
</body>

```
## iframe流

通过在HTML页面里嵌入一个隐藏的iframe,然后将这个iframe的src属性设为对一个长连接的请求,服务器端就能源源不断地往客户推送数据。

```js
let express = require('express')
const app = express()
const path = require('path')
app.get('/clock', function (req, res) {
    // 拿到父窗口，一直往里面写，没有调用end
    setInterval(() => {
      res.write(
        `
        <script>
        parent.document.getElementById('app').innerHTML="${new Date().toLocaleString()}";
        </script>
        `
      )
    }, 1000);
})

app.use(express.stat
```
```js
<body>
  <div id="app">
  </div>
  <iframe src="/clock" frameborder="0"></iframe>
</body>
```
