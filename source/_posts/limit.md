---
title: Node大内存应用
date: 2019-06-30 20:52:55
tags: Node
---

{% asset_img banner.png 图片 %}

在 Node 中，不可避免地还是会存在操作大文件的场景。由于 Node 的内存限制，操作大文件也需要小心，好在 Node 提供了 `stream` 模块用于处理大文件。

<!-- more -->

`stream` 模块是 Node 的原生模块，直接引用即可。 `stream` 继承自 `EventEmitter` ，具备基本的自定义事件功能，同时抽象出标准的事件和方法。它分可读和可写两种。Node中的大多数模块都有 stream 的应用，比如 fs 的 `createReadStream()` 和 `createWriteStream()` 方法可以分别用于创建文件的可读流和可写流， `process` 模块中的 `stdin` 和 `stdout` 则分别是可读流和可写流的示例。

由于V8的内存限制，我们无法通过 `fs.readFile()` 和 `fs.writeFile()` 直接进行大文件的操作，而改用 `fs.createReadStream()` 和 `fs.createWriteStream()` 方法通过流的方式实现对大文件的操作。下面的代码展示了如何读取一个文件，然后将数据写入到另一个文件的过程：

```js
let reader = fs.createReadStream('in.txt');
let writer = fs.createWriteStream('out.txt');
reader.on('data', function (chunk) {
  writer.write(chunk);
});
reader.on('end', function () {
  writer.end();
});
```

由于读写模型固定，上述方法有更简洁的方式，具体如下所示：

```js
let reader = fs.createReadStream('in.txt');
let writer = fs.createWriteStream('out.txt');
reader.pipe(writer);
```

可读流提供了管道方法 `pipe()` ，封装了 `data` 事件和写入操作。通过流的方式，上述代码不会受到V8内存限制的影响，有效地提高了程序的健壮性。如果不需要进行字符串层面的操作，则不需要借助V8来处理，可以尝试进行纯粹的 Buffer 操作，这不会受到V8堆内存的限制。但是这种大片使用内存的情况依然要小心，即使V8不限制堆内存的大小，物理内存依然有限制。
