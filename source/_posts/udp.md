---
title: 构建UDP服务
date: 2019-08-25 10:03:58
tags: HTTP
---
{% asset_img banner.png 图片 %}

UDP又称用户数据包协议，与TCP一样同属于网络传输层。UDP与TCP最大的不同是UDP不是面向连接的。

<!-- more -->

TCP中连接一旦建立，所有的会话都基于连接完成，客户端如果要与另一个TCP服务通信，需要另创建一个套接字来完成连接。

但在UDP中，一个套接字可以与多个UDP服务通信，它虽然提供面向事务的简单不可靠信息传输服务，在网络差的情况下存在丢包严重的问题，但是由于它无须连接，资源消耗低，处理快速且灵活，所以常常应用在那种偶尔丢一两个数据包也不会产生重大影响的场景，比如音频、视频等。UDP目前应用很广泛，DNS服务即是基于它实现的。

node 中的 dgram 模块提供了 UDP 数据包 socket 的实现。创建UDP套接字十分简单，UDP套接字一旦创建，既可以作为客户端发送数据，也可以作为服务器端接收数据。下面的代码创建了一个UDP套接字：

```js
const dgram = require('dgram');
const server = dgram.createSocket('udp4');
server.on('error', (err) => {
console.log(`异常：\n${err.stack}`);
server.close();
});

server.on('message', (msg, rinfo) => {
console.log(`收到${msg}`);
});

server.on('listening', () => {
const address = server.address();
console.log(`监听${address.port}`);
});
server.bind(41234);
// 服务器监听 0.0.0.0:41234
```

只要调用 `dgram.bind(port,  [address])` 方法对网卡和端口进行绑定即可接收网络消息。

```js
socket.send(msg[, offset, length][, port][, address][, callback])
```


在 socket 上广播一个数据包。对于无连接的 socket，必须指定目标的 port 和 address。 


对于连接的 socket，则将会使用其关联的远程端点，因此不能设置 `port` 和 `address` 参数。

这些参数分别为要发送的Buffer、Buffer的偏移、Buffer的长度、目标端口、目标地址、发送完成后的回调。

示例，发送 UDP 包到 localhost 上的某个端口：

```js
const dgram = require('dgram');
const message = Buffer.from('一些字节');
const client = dgram.createSocket('udp4');
client.send(message, 41234, 'localhost', (err) => {
    client.close();
});
```
