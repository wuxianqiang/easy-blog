---
title: 多进程架构
date: 2019-06-30 18:48:50
tags: Node
---

{% asset_img banner.png 图片 %}

CPU的亲和性， 就是进程要在指定的 CPU 上尽量长时间地运行而不被迁移到其他处理器，亲和性是从affinity翻译过来的，应该有点不准确，给人的感觉是亲和性就是有倾向的意思，而实际上是倒向的意思，称为CPU关联性更好，程序员的土话就是绑定CPU，绑核。

<!-- more -->

在多核运行的机器上，每个CPU本身自己会有缓存，缓存着进程使用的信息，而进程可能会被OS调度到其他CPU上，如此，CPU cache命中率就低了，当绑定CPU后，程序就会一直在指定的cpu跑，不会由操作系统调度到其他CPU上，性能有一定的提高。

另外一种使用绑核考虑就是将重要的业务进程隔离开，对于部分实时进程调度优先级高，可以将其绑定到一个指定核上，既可以保证实时进程的调度，也可以避免其他CPU上进程被该实时进程干扰。

## 复制进程

面对单进程单线程对多核使用不足的问题，前人的经验是启动多进程即可。理想状态下每个进程各自利用一个 CPU，以此实现多核CPU的利用。

Node 提供了 child_process 模块，并且也提供了 `child_process.fork()` 函数供我们实现进程的复制。

我们再一次将经典的示例代码存为 `worker.js` 文件，如下所示：

```js
const http = require('http')

const server = http.createServer(
  (req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/plain',
    })
    res.end('hello world')
  }
)

const port = Math.round((1 + Math.random()) * 1000)
server.listen(port, () => {
  console.log(`port in ${port}`)
})
```

通过 `node worker.js` 启动它，将会侦听1000到2000之间的一个随机端口。

将以下代码存为 `master.js`，并通过 `node master.js` 启动它：

```js
const fork = require('child_process').fork
const cpus = require('os').cpus()

for (let i = 0; i < cpus.length; i++) {
  fork('./worker.js');
}
```

这段代码将会根据当前机器上的 CPU 数量复制出对应Node进程数。在控制台会输出对应的端口。

{% asset_img port.png 图片 %}

## 主从模式

著名的 Master-Worker 模式，又称主从模式。图中的进程分为两种：主进程和工作进程。这是典型的分布式架构中用于并行处理业务的模式，具备较好的可伸缩性和稳定性。主进程不负责具体的业务处理，而是负责调度或管理工作进程，它是趋向于稳定的。工作进程负责具体的业务处理，因为业务的多种多样，甚至一项业务由多人开发完成，所以工作进程的稳定性值得开发者关注。

{% asset_img master.png 图片 %}

通过 `fork()` 复制的进程都是一个独立的进程，这个进程中有着独立而全新的 V8 实例。它需要至少30毫秒的启动时间和至少10 MB的内存。尽管 Node 提供了 `fork()` 供我们复制进程使每个CPU内核都使用上，但是依然要切记 `fork()` 进程是昂贵的。好在 Node 通过事件驱动的方式在单线程上解决了大并发的问题，这里启动多个进程只是为了充分将 CPU 资源利用起来，而不是为了解决并发问题。
