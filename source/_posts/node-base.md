---
title: node的基础知识
date: 2019-11-09 09:44:39
tags: Node
---
{% asset_img banner.png 图片 %}

<!-- more -->

## 调用方式

从 JavaScript 调用 Node 的核心模块，核心模块调用C++ 内建模块，内建模块通过 libuv 进行系统调用，这是 Node 里经典的调用方式。

{% asset_img fs.png 图片 %}




## libuv介绍

Node在*nix平台下自行实现了线程池来完成异步I/O。Windows下的IOCP，提供了异步I/O，由于Windows平台和*nix平台的差异，Node提供了libuv作为抽象封装层，使得所有平台兼容性的判断都由这一层来完成，并保证上层的Node与下层的自定义线程池及IOCP之间各自独立。


{% asset_img libuv.png 图片 %}



## 单线程的特点

Node保持了JavaScript在浏览器中单线程的特点。而且在Node中，JavaScript与其余线程是无法共享任何状态的。单线程的最大好处是不用像多线程编程那样处处在意状态的同步问题，这里没有死锁的存在，也没有线程上下文交换所带来的性能上的开销。



单线程的弱点具体有以下3方面。

1. 无法利用多核CPU。
2. 错误会引起整个应用退出，应用的健壮性值得考验。
3. 大量计算占用CPU导致无法继续调用异步I/O。
