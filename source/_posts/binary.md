---
title: 文本文件和二进制文件
date: 2019-06-30 21:16:49
tags: Node
---

{% asset_img banner.png 图片 %}

常见的代码是Node读写文件的操作：

<!-- more -->

```js
const fs = require('fs')

fs.readFile('./a.js', { encoding: 'utf8' }, (err, data) => {
  if (!err) {
    fs.writeFile('./b.js', data, { encoding: 'utf8' }, (err, data) => {
      if (err) {
        console.log('写入失败')
      }
    })
  } else {
    console.log('读取失败')
  }
})

fs.readFile('./a.png', { encoding: 'binary' }, (err, data) => {
  if (!err) {
    fs.writeFile('./b.png', data, { encoding: 'binary' }, (err, data) => {
      if (err) {
        console.log('写入失败')
      }
    })
  } else {
    console.log('读取失败')
  }
})
```

你是否注意到上面代码的区别，上面有个 utf8 和 binary 的单词，作用是区分文本文件和二进制文件，具体什么时候该用文本文件，又什么时候用二进制文件呢？那么就是今天要介绍的内容。

文本文件和二进制文件从本质上来说他们之间没有什么区别，因为他们在硬盘上都二进制存放的。

文本文件是一种由很多行字符构成的计算机文件。文本文件存在于计算机系统中，通常在文本文件最后一行放置文件结束标志。文本文件的编码基于字符定长，译码相对要容易一些；二进制文件编码是变长的，灵活利用率要高，而译码要难一些，不同的二进制文件译码方式是不同的。

html,css,js文件都是文本文件，需要设置uft8的文本模式，而图片,音频,视频都是二进制文件。需要设置binary的二进制模式，设置了文件的编码方式之后才能保证文件打开的时候不乱码。

二进制文件是计算机需要通过特定的软件处理分析才能显示的，比如打开图片需要使用查看图片的软件。关于图像识别的知识我就不在里面介绍了。
