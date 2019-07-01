---
title: Node自动重启
date: 2019-06-30 18:10:00
tags: Node
---

{% asset_img banner.png 图片 %}

`child_process.fork()` 方法是专门用于衍生新的 Node.js 进程。 返回 ChildProcess  类  ，ChildProcess 类的实例都是 EventEmitter，表示衍生的子进程。

<!-- more -->

```js
const fork = require('child_process').fork;
const childProcess  = fork('./worker.js')
```

当子进程使用 `process.send()` 发送消息时会触发 `'message'` 事件。

```js
process.on('message', (message, sendHandle) => {
	// TODO
})
```

当子进程结束后时会触发 `'exit'` 事件。 如果进程退出，则 code 是进程的最终退出码，否则为 null。 如果进程是因为收到的信号而终止，则 signal 是信号的字符串名称，否则为 null。 这两个值至少有一个是非空的。

```js
childProcess.on('exit', (code, signal) => {
  // TODO
})
```

## 自动重启

有了父子进程之间的相关事件之后，就可以在这些关系之间创建出需要的机制了。至少我们能够通过监听子进程的 `exit` 事件来获知其退出的信息，接着前文的多进程架构，我们在主进程上要加入一些子进程管理的机制，比如重新启动一个工作进程来继续服务。

```js
// worker.js 子进程文件
const http = require('http')
const process = require('process')

const server = http.createServer(
  (req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/plain',
    })
    res.end('handle by child')
  }
)

process.on('message', (m, tcp) => {
  if (m === 'server') {
    tcp.on('connection', socket => {
      server.emit('connection', socket)
    })
  }
})
```
```js
// master.js 主进程文件
const fork = require('child_process').fork;
const cpus = require('os').cpus();
const net = require('net')
const server = net.createServer()

server.listen(8080)

const workers = {}
const createWorker = function () {
  const worker  = fork('./worker.js')
  worker.on('exit', () => {
    console.log(`进程退出：${worker.pid}`)
    delete workers[worker.pid]
    createWorker()
  })
  worker.send('server', server)
  workers[worker.pid] = worker
  console.log(`进程创建：${worker.pid}`)
}

for (let i = 0; i < cpus.length; i++) {
  createWorker()
}
```

测试一下上面的代码，在 window 电脑中尝试使用 `taskkill /pid 524 /f` 杀死进程号为 `524` 的进程，发现会自动重启一个 `6976` 的进程。总体进程数量并没有发生改变。

{% asset_img process3.png 图片 %}

在这个场景中我们主动杀死了一个进程，在实际业务中，可能有隐藏的bug导致工作进程退出，那么我们需要仔细地处理这种异常。

NodeJS 对于未捕获异常的默认处理是：触发 `uncaughtException` 事件，如果 `uncaughtException` 没有被监听，那么打印异常的堆栈信息，触发进程的 `exit` 事件。

```js
const http = require('http')
const process = require('process')

const server = http.createServer(
  (req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/plain',
    })
    res.end('handle by child')
  }
)

let worker;
process.on('message', (m, tcp) => {
  if (m === 'server') {
    worker = tcp;
    tcp.on('connection', socket => {
      server.emit('connection', socket)
    })
  }
})

// 监听未捕获的异常
process.on('uncaughtException', () => {
  // 停止接收新的连接
  worker.close(() => {
    // 已有连接断开后退出进程
    process.exit(1)
  })
})
```

上述代码的处理流程是，一旦有未捕获的异常出现，工作进程就会立即停止接收新的连接；当所有连接断开后，退出进程。主进程在侦听到工作进程的 `exit` 后，将会立即启动新的进程服务，以此保证整个集群中总是有进程在为用户服务的。

{% asset_img process2.png 图片 %}

## 自杀信号

当然上述代码存在的问题是要等到已有的所有连接断开后进程才退出，在极端的情况下，所有工作进程都停止接收新的连接，全处在等待退出的状态。但在等到进程完全退出才重启的过程中，所有新来的请求可能存在没有工作进程为新用户服务的情景，这会丢掉大部分请求。

为此需要改进这个过程，不能等到工作进程退出后才重启新的工作进程。当然也不能暴力退出进程，因为这样会导致已连接的用户直接断开。于是我们在退出的流程中增加一个自杀（suicide）信号。工作进程在得知要退出时，向主进程发送一个自杀信号，然后才停止接收新的连接，当所有连接断开后才退出。主进程在接收到自杀信号后，立即创建新的工作进程服务。代码改动如下所示：

```js
const fork = require('child_process').fork;
const cpus = require('os').cpus();
const net = require('net')
const server = net.createServer()

server.listen(8080)

const workers = {}
const createWorker = function () {
  const worker  = fork('./worker.js')
  worker.on('exit', () => {
    console.log(`进程退出：${worker.pid}`)
    delete workers[worker.pid]
    createWorker()
  })
  // 创建一个新的进程
  worker.on('message', message => {
    if (message.act === 'suicide') {
      createWorker()
    }
  })
  worker.send('server', server)
  workers[worker.pid] = worker
  console.log(`进程创建：${worker.pid}`)
}

for (let i = 0; i < cpus.length; i++) {
  createWorker()
}
```

```js
const http = require('http')
const process = require('process')

const server = http.createServer(
  (req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/plain',
    })
    res.end('handle by child')
  }
)

let worker;
process.on('message', (m, tcp) => {
  if (m === 'server') {
    worker = tcp;
    tcp.on('connection', socket => {
      server.emit('connection', socket)
    })
  }
})

// 监听未捕获的异常
process.on('uncaughtException', () => {
  // 通知主进程去创建一个新的进程
  process.send({act: 'suicide'})
  // 停止接收新的连接
  worker.close(() => {
    // 所有已有连接断开后退出进程
    process.exit(1)
  })
})
```

至此我们完成了进程的平滑重启，一旦有异常出现，主进程会创建新的工作进程来为用户服务，旧的进程一旦处理完已有连接就自动断开。整个过程使得我们的应用的稳定性和健壮性大大提高。

{% asset_img process2.png 图片 %}

这里存在问题的是有可能我们的连接是长连接，不是HTTP服务的这种短连接，等待长连接断开可能需要较久的时间。为此为已有连接的断开设置一个超时时间是必要的，在限定时间里强制退出的设置如下所示：

```js
const http = require('http')
const process = require('process')

const server = http.createServer(
  (req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/plain',
    })
    res.end('handle by child')
  }
)

let worker;
process.on('message', (m, tcp) => {
  if (m === 'server') {
    worker = tcp;
    tcp.on('connection', socket => {
      server.emit('connection', socket)
    })
  }
})

// 监听未捕获的异常
process.on('uncaughtException', () => {
  // 通知主进程去创建一个新的进程
  process.send({act: 'suicide'})
  // 停止接收新的连接
  worker.close(() => {
    // 所有已有连接断开后退出进程
    process.exit(1)
  })
  // 5秒后退出进程
  setTimeout(() => {
    process.exit(1)
  }, 5000)
})
```
## 限量重启

通过自杀信号告知主进程可以使得新连接总是有进程服务，但是依然还是有极端的情况。工作进程不能无限制地被重启，如果启动的过程中就发生了错误，或者启动后接到连接就收到错误，会导致工作进程被频繁重启，这种频繁重启不属于我们捕捉未知异常的情况，因为这种短时间内频繁重启已经不符合预期的设置，极有可能是程序编写的错误。

为了消除这种无意义的重启，在满足一定规则的限制下，不应当反复重启。比如在单位时间内规定只能重启多少次，超过限制就触发 `giveup` 事件，告知放弃重启工作进程这个重要事件。

为了完成限量重启的统计，我们引入一个队列来做标记，在每次重启工作进程之间进行打点并判断重启是否太过频繁，如下所示：

```js
const fork = require('child_process').fork;
const cpus = require('os').cpus();
const net = require('net')
const server = net.createServer()
server.listen(8080)

// 重启次数
const limit = 10;
// 时间单位
const during = 60000;
let restart = [];

const isTooFrequently = function () {
  // 记录重启时间
  const time = Date.now();
  const length = restart.push(time);
  if (length > limit) {
    // 取出最后10个记录
    restart = restart.slice(limit * -1);
  }
  // 最后一次重启到前10次重启之间的时间间隔
  const step = restart[restart.length - 1] - restart[0];
  return restart.length >= limit && step < during
};

const workers = {}
const createWorker = function () {
  // 检查是否太过频繁
  if (isTooFrequently()) {
    // 触发giveup事件后，不再重启
    process.emit('giveup', length, during);
    return;
  }
  const worker = fork('./worker.js')
  worker.on('exit', () => {
    console.log(`进程退出：${worker.pid}`)
    delete workers[worker.pid]
    createWorker()
  })
  worker.on('message', message => {
    if (message.act === 'suicide') {
      createWorker()
    }
  })
  worker.send('server', server)
  workers[worker.pid] = worker
  console.log(`进程创建：${worker.pid}`)
}

for (let i = 0; i < cpus.length; i++) {
  createWorker()
}
```
`giveup` 事件是比 `uncaughtException` 更严重的异常事件。 `uncaughtException` 只代表集群中某个工作进程退出，在整体性保证下，不会出现用户得不到服务的情况，但是这个 `giveup` 事件则表示集群中没有任何进程服务了，十分危险。为了健壮性考虑，我们应在 `giveup` 事件中添加重要日志，并让监控系统监视到这个严重错误，进而报警等。
