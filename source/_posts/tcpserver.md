---
title: 构建TCP服务
date: 2019-08-04 08:39:32
tags: HTTP
---
{% asset_img banner.png 图片 %}

TCP服务在网络应用中十分常见，目前大多数的应用都是基于TCP搭建而成的。

<!-- more -->

TCP全名为传输控制协议，在OSI模型（由七层组成，分别为物理层、数据链结层、网络层、传输层、会话层、表示层、应用层）中属于传输层协议。许多应用层协议基于TCP构建，典型的是HTTP、SMTP、IMAP等协议。七层协议示意图如图所示。

{% asset_img tcp.png 图片 %}

TCP是面向连接的协议，其显著的特征是在传输之前需要3次握手形成会话。

只有会话形成之后，服务器端和客户端之间才能互相发送数据。在创建会话的过程中，服务器端和客户端分别提供一个套接字，这两个套接字共同形成一个连接。服务器端与客户端则通过套接字实现两者之间连接的操作。

创建TCP服务器
在基本了解TCP的工作原理之后，我们可以开始创建一个TCP服务器端来接受网络请求，代码如下：


```js
const net = require('net');

const server = net.createServer((socket) => {
  console.log('客户端已连接')
  socket.on('end', () => {
    console.log('客户端已断开')
  })
  socket.write('你好\n')
  socket.pipe(c)
})

server.listen(3000, () => {
  console.log('服务器已启动')
}
```

通过 net 模块自行构造客户端进行会话，测试上面构建的TCP服务的代码如下所示：


```js
const net = require('net');
const client = net.createConnection({port: 3000}, () => {
  console.log('连接到服务器')
  client.write('world\r\n');
})
client.on('data', (data) => {
  console.log(data.toString())
  client.end();
})
client.on('en', () => {
  console.log('断开了服务器')
}
```

由于TCP套接字是可写可读的 Stream 对象，可以利用 pipe() 方法巧妙地实现管道操作。



在Node中，由于TCP默认启用了Nagle算法，可以调用 socket.setNoDelay(true) 去掉Nagle算法，使得 write() 可以立即发送数据到网络中。



另一个需要注意的是，尽管在网络的一端调用 `write()` 会触发另一端的 `data` 事件，但是并不意味着每次 `write()` 都会触发一次 `data` 事件，在关闭掉 Nagle算法后，另一端可能会将接收到的多个小数据包合并，然后只触发一次 `data` 事件。
