---
title: Buffer对象的内存分配
date: 2019-06-30 20:15:49
tags: Node
---

{% asset_img banner.jpg 图片 %}

Buffer对象的内存分配不是在V8的堆内存中，而是在Node的C++层面实现内存的申请的。因为处理大量的字节数据不能采用需要一点内存就向操作系统申请一点内存的方式，这可能造成大量的内存申请的系统调用，对操作系统有一定压力。为此Node在内存的使用上应用的是在C++层面申请内存、在JavaScript中分配内存的策略。

<!-- more -->

为了高效地使用申请来的内存，Node采用了slab分配机制。slab是一种动态内存管理机制简单而言，slab就是一块申请好的固定大小的内存区域。slab具有如下3种状态。

1. full：完全分配状态。
2. partial：部分分配状态。
3. empty：没有被分配状态。

```js
const buf = Buffer.alloc(5);
console.log(buf);
// 打印: <Buffer 00 00 00 00 00>
```

通过 `Buffer.alloc` 来分配一个大小为5字节的新 Buffer。

Node以8 KB为界限来区分Buffer是大对象还是小对象这个8  KB的值也就是每个slab的大小值，在JavaScript层面，以它作为单位单元进行内存的分配。

如果指定Buffer的大小少于8 KB，Node会按照小对象的方式进行分配。Buffer的分配过程中主要使用一个局部变量pool 作为中间处理对象，处于分配状态的slab单元都指向它。以下是分配一个全新的slab单元的操作，它会将新申请的 SlowBuffer 对象指向它：

```js
let pool;
function allocPool() {
  pool = new SlowBuffer(Buffer.poolSize);
  pool.used = 0;
}
```

新构造的slab单元slab处于empty状态。从一个新的slab单元中初次分配一个Buffer对象这时候的slab状态为partial。

{% asset_img buffer.png 图片 %}

当再次创建一个Buffer对象时，构造过程中将会判断这个slab的剩余空间是否足够。如果足够，使用剩余空间，并更新slab的分配状态。如果slab剩余的空间不够，将会构造新的slab，原slab中剩余的空间会造成浪费。

{% asset_img buffer2.png 图片 %}

如果需要超过8 KB的Buffer对象，将会直接分配一个 SlowBuffer 对象作为slab单元，这个slab单元将会被这个大Buffer对象独占。

简单而言，真正的内存是在Node的C++层面提供的，JavaScript层面只是使用它。当进行小而频繁的Buffer操作时，采用slab的机制进行预先申请和事后分配，使得JavaScript到操作系统之间不必有过多的内存申请方面的系统调用。对于大块的Buffer而言，则直接使用C++层面提供的内存，而无需细腻的分配操作。
