---
title: 简单入门buffer
date: 2019-06-30 18:59:45
tags: Node
---

{% asset_img banner.png 图片 %}

如果你第一次认识buffer，你可能会很陌生，因为在前端的JavaScript中并没有buffer，因为前端只要做一些字符串操作或DOM基本操作就能满足业务需求。

<!-- more -->

# buffer是什么？

buffer是Node底层通过C++申请的内存，通过JS来分配内存。也就是存放文件的缓冲区。那么问题来了，为什么叫做缓存区，了解之前就要先跟大家科普一下V8的内存限制。

当我们在代码中声明变量并赋值时，所使用对象的内存就分配在堆中。如果已申请的堆空闲内存不够分配新的对象，将继续申请堆内存，直到堆的大小超过V8的限制为止。（64位系统下约为1.4 GB，32位系统下约为0.7 GB）

那么又为什么要限制，其实这样设计是有目的，方便V8做垃圾回收，在做垃圾回收时JS是停滞的，等待垃圾回收完成再恢复，所以垃圾回收的耗时关系到我们的性能，当内存太大就耗时越长，所以V8要这么限制。

在这样的限制下，将会导致Node无法直接操作大内存对象，比如无法将一个2 GB的文件读入内存中进行字符串分析处理，那么我们就将文件读取到buffer中，暂时存放，所以叫做缓冲区。

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
```
不要使用 `fs.readFiles` 和 `fs.writeFile` 直接进行大文件操作，而是使用 `fs.createReadStream` 和 `fs.createWriteStream` 方法通过流的方式实现对大文件的操作.
```JS
let reader = fs.createReadStream('a.js');
let writer = fs.createWriteStream('b.js');
reader.pipe(writer);
```
通过流的方式操作文件里面的原理也就是使用了 buffer 作为缓冲。每次读一点写一点。

**虽然buffer是申请的内存，不受V8内存的限制，但是物理内存依然是有限的。**

# 了解Buffer

Buffer是一个像Array的对象，但它主要用于**操作字节**。

由于Buffer太过常见，Node在进程启动时就已经加载了它，并将其放在全局对象（global）上。所以在使用Buffer时，无须通过 `require()`  即可直接使用。

Buffer对象类似于数组，它的元素为16进制的两位数，即0到255的数值。

```js
let buf = Buffer.from('hello', 'utf8')
console.log(buf)
// <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
console.log(buf.length) // 5
console.log(buf[0]) // 104
```

可以访问 `length` 属性得到长度(表示的是字节的长度，在utf8编码中汉字是3字节，英文是1字节)，也可以通过下标访问元素。

关于下标访问元素需要注意的是要经历一个计算的过程。我们通过获取第一个字母为例，首先是 `h` 根据ASCII码对照表查出来16进制是68，然后通过JS中的 `toString` 方法传入参数为2表示转换成二进制，得到 `1101000`，然后再换成10进制的数就变成了104，所以通过下标访问的第一个元素结果是104，具体代码如下。

```js
let buf = Buffer.from('hello', 'utf8')
console.log(buf)
// <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
console.log((0x68).toString(2))
// 1101000
```

`Buffer.from` 的用法在这里介绍一下，第一个参数是字符串，表示创建一个包含字符串的buffer，第二个参数是指定字符的编码，默认值也是 `utf8`。
