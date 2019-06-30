---
title: Node中POST请求的正确处理方式
date: 2019-06-30 18:38:48
tags: Node
---
{% asset_img banner.jpg 图片 %}

Node的 http 模块只对HTTP报文的头部进行了解析，然后触发 `request` 事件。如果请求中还带有内容部分（如 POST 请求，它具有报头和内容），内容部分需要用户自行接收和解析。

<!-- more -->

通过报头的 `Transfer-Encoding` 或 `Content-Length` 即可判断请求中是否带有内容

| 字段名称 | 含义 |
|--|--|
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Content-Length | 报文主体的大小 |

写个方法判断是否有报文主体
```js
const hasBody = function(req) {
  return  'transfer-encoding'  in  req.headers  ||  'content-length' in req.headers;
};
```
# 接收数据
报文内容部分会通过 `data` 事件触发，我们只需以流的方式处理即可，不要在订阅 `data` 事件的时候使用 `+=` 的形式拼装数据，这样会乱码的。

```js
function handle(req, res) {
  if (hasBody(req)) {
    var buffers = [];
    req.on('data', function (chunk) {
      buffers.push(chunk);
    });
    req.on('end', function () {
      const POST = Buffer.concat(buffers).toString();
    });
  }
}
```
## 1. POST发送的是表单的数据
如果在页面中使用表单提交一个post请求，我们的代码大概是这样的。
```js
  <form action="/upload" method="post">
    <label for="username">用户名:</label>
    <input type="text" name="username" id="username" />
    <label for="password">密码:</label>
    <input type="password" name="password" id="password" />
    <input type="submit" />
  </form>
```
默认的表单提交，请求头中的 `Content-Type` 字段值为 `application/x-www-form-urlencoded`
```
Content-Type: application/x-www-form-urlencoded
```

写一个判断内容类型的方法
```js
const mime = function (req) {
  const str = req.headers['content-type'] || '';
  return str.split(';')[0];
};
```

它的报文体内容跟查询字符串相同
```
username=Tom&password=123456
```
解析表单数据使用`querystring`模块中的`parse`方法
```js
const querystring = require('querystring')
function handleForm (req, res) {
  const isFrom = mime(req) === 'application/x-www-form-urlencoded'
  if (hasBody(req) && isFrom) {
    var buffers = [];
    req.on('data', function (chunk) {
      buffers.push(chunk);
    });
    req.on('end', function () {
      let requestBody = Buffer.concat(buffers).toString();
      requestBody = querystring.parse(requestBody)
    });
  }
}
```
## 2. POST发送的是JSON的数据
如果在页面中使用axios发送post请求，我们的代码大概是这样的。
```js
axios.post('/user', {
  username: 'Tom',
  password: '123456'
})
```
默认的JSON提交，请求头中的 `Content-Type` 字段值为 `application/json`，在 `Content-Type` 中可能还附带编码信息 `charset=utf-8`
```
Content-Type: application/json; charset=utf-8
```
它的报文体内容跟JSON格式的字符串相同
```js
{
  "name": "Tom",
  "password": "123456"
}
```
解析JSON数据使用 `JSON.parse` 方法。
```js
function handleJson (req, res) {
  const isJson = mime(req) === 'application/json'
  if (hasBody(req) && isJson) {
    var buffers = [];
    req.on('data', function (chunk) {
      buffers.push(chunk);
    });
    req.on('end', function () {
      let requestBody = Buffer.concat(buffers).toString();
      try {
        requestBody = JSON.parse(requestBody)
      } catch (error) {
        console.log(error)
      }
    });
  }
}
```
## 3. POST发送的是文件数据
如果在页面中使用表单提交文件请求，我们的代码大概是这样的。
```js
  <form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="avatar" id="avatar">
    <input type="submit" />
  </form>
```
默认的上传文件提交，请求头中的 `Content-Type` 字段值为`multipart/form-data`，在 `Content-Type` 中可能还附带内容分隔符 `boundary=----WebKitFormBoundary4Hsing01Izo2AHqv`
```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary4Hsing01Izo2AHqv
```
先上传一个JS文件，看看报文主体里面的格式大概是这样的，包含文件信息和文件内容，有指定的分隔符包裹。

{% asset_img post.png 图片 %}

上传文件的时候是要区分文本文件和二进制文件，文本文件是要使用 `utf8` 编码(HTML，CSS，JavaScript)，二进制文件是要使用 `binary` 编码(图片，视频，音频)

根据内容分隔符解析上传的图片，并且写入到文件中，下面代码暂时只处理图片格式的文件。
```js
function handleFile(req, res) {
  const isFile = mime(req) === 'multipart/form-data'
  if (hasBody(req) && isFile) {
    var buffers = [];
    req.on('data', function (chunk) {
      buffers.push(chunk);
    });
    req.on('end', function () {
      // 处理文件名
      let requestBody = Buffer.concat(buffers).toString('binary');
      let file = querystring.parse(requestBody, '\r\n', ': ')
      let fileInfo = file['Content-Disposition']
      fileInfo = Buffer.from(fileInfo, 'binary').toString()
      let { filename } = querystring.parse(fileInfo, '; ', '=')
      filename = filename.slice(1, -1)
      filename = `./static/${filename}`
      // 处理内容
      let boundary = req.headers['content-type'].split('; ')[1].replace('boundary=', '');
      let contentType = file['Content-Type']
      if (!contentType.includes('image')) return
      let upperBoundary = requestBody.indexOf(contentType) + contentType.length;
      let shorterData = requestBody.substring(upperBoundary)
      let binaryDataAlmost = shorterData.trim()
      let binaryData = binaryDataAlmost.substring(0, binaryDataAlmost.indexOf(`--${boundary}--`))
      // 写入文件
      fs.writeFile(filename, binaryData, 'binary', (err) => {
        if (err) {
          console.log('上传失败')
        } else {
          console.log('上传成功', filename)
        }
      })
    });
  }
}
```

这就是所有处理POST请求的方式，你都学会了吗？
