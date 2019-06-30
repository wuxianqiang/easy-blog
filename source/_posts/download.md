---
title: 附件下载原来如此简单
date: 2019-06-30 21:30:41
tags: Node
---
{% asset_img banner.png 图片 %}

之前我们已经了解过了 POST 上传的数据处理，以及包括上传的图片怎么处理，那么文件下载又是一个怎么样的过程呢？今天就要来好好说道说道。

<!-- more -->

MIME 类型
不同的文件还要设置不同的 MIME 类型，具体什么是 MIME 类型呢？其他只是一种文件类型的标识，设置了之后浏览器才能按照指定的内容显示。

| MIME类型 | 文本类型 |
| -- | -- |
| application/javascript | javascript |
| application/json | json |
| text/html | html |
| text/css | css |
| image/gif | GIF 图片 (无损耗压缩方面被PNG所替代) |
| image/jpeg | JPEG 图片 |
| image/png | PNG 图片 |
| image/svg+xml | SVG图片 (矢量图) |

关于更多文件的 MIME 的介绍请看MDN[1]。

## Content-Disposition 字段

无论响应的内容是什么样的 MIME 值，需求中并不要求客户端去打开它，只需弹出并下载它即可。为了满足这种需求， `Content-Disposition` 字段应声登场。

`Content-Disposition` 字段影响的行为是客户端会根据它的值判断是应该将报文数据当做即时浏览的内容，还是可下载的附件。

当内容只需即时查看时，它的值为 `inline` ，当数据可以存为附件时，它的值为 `attachment` 。

另外， `Content-Disposition` 字段还能通过参数指定保存时应该使用的文件名。示例如下：

```js
Content-Disposition: attachment; filename="filename.ext"
```

关于更多文报文头字段的介绍请看MDN[2]。

## 下载图片

知道字段以后。我们通过 Node 搭建一个 HTTP 服务，使用文件模块中的 `stat` 方法读取文件中图片，这个方法会返回一些文件的信息，包括文件的大小，通过 `Content-Length` 设置文件的大小。

```js
const http = require('http')
const url = require('url')
const fs = require('fs')
const server = http.createServer((req, res) => {
  const { pathname } = url.parse(req.url)
  if (pathname === '/download') {
    fs.stat('./images/th.jpg', function (err, stat) {
      const stream = fs.createReadStream('./images/th.jpg')
      res.writeHead(200, {
        'Content-Type': 'image/jpeg',
        'Content-Length': stat.size,
        'Content-Disposition': 'attachment; filename=th.jpg'
      })
      stream.pipe(res)
    });
    return
  }
  res.end('404')
})

server.listen(8080, () => {
  console.log('port in 8080')
})
```

上面代码中还使用到了一个 `writeHead` 方法，这个方法就是设置响应头的，第一个参数是 HTTP 的状态码数字，第二个参数是个HTTP原因短语字符串，比如 `ok`，这个参数可以省了，第三个参数是设置响应头字段的对象。
