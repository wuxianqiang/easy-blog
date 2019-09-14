---
title: 请求头字段
date: 2019-09-14 17:41:56
tags: HTTP
---

{% asset_img banner.png 图片 %}

<!-- more -->

HTTP 首部字段是由首部字段名和字段值构成的，中间用冒号“:” 分隔。


```
首部字段名: 字段值
```

另外，字段值对应单个 HTTP 首部字段可以有多个值，多个指令之间通过“,”分隔。如下所示。

```
Keep-Alive: timeout=15, max=100
```

若想要加优先级，则使用 q=来额外表示权重值  1 ，用分号（;）进行分隔。权重值 q 的范围是 0~1（可精确到小数点后 3位），且 1 为最大值。不指定权重 q 值时，默认权重为 q=1.0。

```
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
```

## 请求头字段

```
Accept: image/webp,image/apng,image/*,*/*;q=0.8
```

Accept 首部字段可通知服务器，用户代理能够处理的媒体类型及媒体类型的相对优先级。可使用type/subtype 这种形式，一次指定多种媒体类型。

```
Accept-Encoding: gzip, deflate, br
```

Accept-Encoding 首部字段用来告知服务器用户代理支持的内容编码及内容编码的优先级顺序。可一次性指定多种内容编码。

```
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
```

首部字段 Accept-Language 用来告知服务器用户代理能够处理的自然语言集（指中文或英文等），以及自然语言集的相对优先级。可一次指定多种自然语言集。

```
Cache-Control: no-cache
```

客户端发送的请求中如果包含 no-cache 指令，则表示客户端将不会接收缓存过的响应。于是，“中间”的缓存服务器必须把客户端请求转发给源服务器。

```
Connection: keep-alive
```

HTTP/1.1 之前的 HTTP 版本的默认连接都是非持久连接。为此，如果想在旧版本的HTTP 协议上维持持续连接，则需要指定Connection 首部字段的值为 Keep-Alive。

```
Cookie: PSTM=1568244881
```

首部字段 Cookie 会告知服务器，当客户端想获得HTTP 状态管理支持时，就会在请求中包含从服务器接收到的 Cookie。

```
Host: www.baidu.com
```

首部字段 Host 会告知服务器，请求的资源所处的互联网主机名和端口号。Host 首部字段在HTTP/1.1 规范内是唯一一个必须被包含在请求内的首部字段。

```
Referer: https://www.baidu.com/
```

首部字段 Referer 会告知服务器请求的原始资源的 URI。

```
Pragma: no-cache
```

Pragma 是 HTTP/1.1 之前版本的历史遗留字段，仅作为与 HTTP/1.0 的向后兼容而定义。客户端会要求所有的中间服务器不返回缓存的资源。

```
User-Agent: Mozilla/5.0 (Windows NT 6.1;)
```

首部字段 User-Agent 会将创建请求的浏览器和用户代理名称等信息传达给服务器。
