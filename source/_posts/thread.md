---
title: 多线程编程
date: 2019-11-09 09:40:27
tags: Node
---
{% asset_img banner.jpg 图片 %}



我们在谈论JavaScript的时候，通常谈的是单一线程上执行的代码，这在浏览器中指的是JavaScript执行线程与UI渲染共用的一个线程；

<!-- more -->

在Node中，只是没有UI渲染的部分，模型基本相同。对于服务器端而言，如果服务器是多核CPU，单个Node进程实质上是没有充分利用多核CPU的。



随着现今业务的复杂化，对于多核CPU利用的要求也越来越高。



浏览器提出了Web  Workers，它通过将JavaScript执行与UI渲染分离，可以很好地利用多核CPU为大量计算服务。同时前端Web Workers也是一个利用消息机制合理使用多核CPU的理想模型。

{% asset_img thread.png 图片 %}




遗憾在于前端浏览器存在对标准的滞后性，Web Workers并没有广泛应用起来。



另外Web Workers能解决利用CPU和减少阻塞UI渲染，但是不能解决UI渲染的效率问题。Node借鉴了这个模式， child_process 是其基础API， cluster 模块是更深层次的应用。



借助Web Workers的模式，开发人员要更多地去面临跨线程的编程，这对于以往的JavaScript编程经验是较少考虑的。



习惯异步编程的同学，也许能够从容面对异步编程带来的副产品，比如嵌套回调、业务分散等问题。但对于异步调用，通过良好的流程控制，还是能够将逻辑梳理成顺序式的形式。（发布订阅，Promise形式等）
