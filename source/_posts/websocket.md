---
title: 构建websocket服务
date: 2019-08-25 10:00:29
tags: HTTP
---

{% asset_img banner.png 图片 %}

<!-- more -->

提到Node，不能错过的是WebSocket协议。它与Node之间的配合堪称完美，其理由有两条。

No.1

WebSocket客户端基于事件的编程模型与Node中自定义事件相差无几。

No.2

WebSocket实现了客户端与服务器端之间的长连接，而Node事件驱动的方式十分擅长与大量的客户端保持高并发连接。

除此之外，WebSocket与传统HTTP有如下好处。

No.1

客户端与服务器端只建立一个TCP连接，可以使用更少的连接。

No.2

WebSocket服务器端可以推送数据到客户端，这远比HTTP请求响应模式更灵活、更高效。

No.3

有更轻量级的协议头，减少数据传送量。

现代浏览器大多都支持WebSocket协议，接下来我们用一段代码来展现WebSocket在客户端的应用示例：

```js
let socket = new WebSocket('ws://127.0.0.1:8080/updates');
socket.onopen = function () {
  setInterval(function () {
    if (socket.bufferedAmount == 0) socket.send(getUpdateData());
  }, 50);
};
socket.onmessage = function (event) {
  // TODO：event.data
};
```

上述代码中，浏览器与服务器端创建WebSocket协议请求，在请求完成后连接打开，每50毫秒向服务器端发送一次数据，同时可以通过 onmessage()方法接收服务器端传来的数据。这行为与TCP客户端十分相似，相较于HTTP，它能够双向通信。浏览器一旦能够使用WebSocket，可以想象应用的使用空间极大。

在WebSocket之前，网页客户端与服务器端进行通信最高效的是Comet技术。实现Comet技术的细节是采用长轮询（long-polling）或iframe流。长轮询的原理是客户端向服务器端发起请求，服务器端只在超时或有数据响应时断开连接（ res.end() ）；客户端在收到数据或者超时后重新发起请求。

使用WebSocket的话，网页客户端只需一个TCP连接即可完成双向通信，在服务器端与客户端频繁通信时，无须频繁断开连接和重发请求。连接可以得到高效应用，编程模型也十分简洁。

前文也或多或少提到了WebSocket与HTTP的区别，相比HTTP，WebSocket更接近于传输层协议，它并没有在HTTP的基础上模拟服务器端的推送，而是在TCP上定义独立的协议。让人迷惑的部分在于WebSocket的握手部分是由HTTP完成的，使人觉得它可能是基于HTTP实现的。

WebSocket协议主要分为两个部分：握手和数据传输。之后我们来详细说一说这两个部分。
