---
title: express基本原理
date: 2019-06-30 21:24:05
tags: Node
---

{% asset_img banner.png 图片 %}

了解 express 原理之前，你需要先掌握 express 的基本用法。

<!-- more -->

关于 express 的介绍请看 [express 官网](http://expressjs.com/)。

## 基本结构

先回顾一下 express 使用的的过程，首先是把模块倒入，然后当做方法执行，在返回值中调用 `use` 处理路由，调用 `listen` 监听端口。

```js
const express = require('express')
const app = express()
app.use('/home', (req, res) => {
  res.end('home')
})
app.listen(8080, () => {
  console.log('port created successfully')
})
```


根据上面的使用，我们开始构建代码。我们需要写一个 `express` 方法，返回一个 `app` 对象，有 `use` 和 `listen` 方法。


```js
const http = require('http')
const url = require('url')

function express() {
  const app = {}
  const routes = [];
  
  app.use = function (path, action) {
    routes.push([path, action])
  }

  function handle(req, res) {
    let pathname = url.parse(req.url).pathname;
    for (let i = 0; i < routes.length; i++) {
      var route = routes[i];
      if (pathname === route[0]) {
        let action = route[1];
        action(req, res);
        return;
      }
    }
    handle404(req, res);
  }

  function handle404(req, res) {
    res.end('404')
  }

  app.listen = function (...args) {
    const server = http.createServer((req, res) => {
      handle(req, res)
    })
    server.listen(...args)
  }

  return app
}

module.exports = express
```

上面代码中的 `use` 方法的作用是把请求路径跟对应的处理函数存放在一个数组中，当请求到来的时候遍历数组，根据路径找到对应的方法执行。

## 动态路由
动态路由是根据参数可以动态匹配路径。

```js
const express = require('./express')
const app = express()

// /home/1
// /home/2
app.use('/home/:id', (req, res) => {
  res.end('home')
})

app.listen(8080, () => {
  console.log('port created successfully')
})
```

根据路由里面的参数要匹配符合规则的路由我们需要使用正则来处理，下面代码是根据路径来生成正则的一个方法。

```js
const pathRegexp = (path, paramNames=[], {end=false} ={}) => {
  path = path
    .concat(end ? '' : '/?')
    .replace(/\/\(/g, '(?:/')
    .replace(/(\/)?(\.)?:(\w+)(?:(\(.*?\)))?(\?)?(\*)?/g, function (_, slash, format, key, capture, optional, star) {
      slash = slash || '';
      paramNames.push(key);
      return ''
        + (optional ? '' : slash)
        + '(?:'
        + (optional ? slash : '')
        + (format || '') + (capture || (format && '([^/.]+?)' || '([^/]+?)')) + ')'
        + (optional || '')
        + (star ? '(/*)?' : '');
    })
    .replace(/([\/.])/g, '\\$1')
    .replace(/\*/g, '(.*)');
  return new RegExp('^' + path + '$')
}

module.exports = pathRegexp
```

根据路径生成正则也是有第三方模块 [path-to-regexp]( https://github.com/pillarjs/path-to-regexp) 模块，核心原理大家值得参考。包括 Vue 和 React 的路由都使用到了这个模块。

下面我们需要开始动态映射路由。

```js
const http = require('http')
const url = require('url')
const pathRegexp = require('./pathRegexp')

function express() {
  const app = {}
  const routes = { 'all': [] };

  app.use = function (path, action) {
    const keys = []
    const regexp = pathRegexp(path, keys,{end:true})
    routes.all.push([
      { regexp, keys },
      action
    ]);
  };

  ['get', 'put', 'delete', 'post'].forEach(function (method) {
    routes[method] = [];
    app[method] = function (path, action) {
      const keys = []
      const regexp = pathRegexp(path, keys, {end:true})
      routes[method].push([
        { regexp, keys },
        action
      ]);
    };
  });

  const match = function (pathname, routes, req, res) {
    for (var i = 0; i < routes.length; i++) {
      let route = routes[i];
      let reg = route[0].regexp;
      let keys = route[0].keys;
      let matched = reg.exec(pathname);
      if (matched) {
        let params = {};
        for (let i = 0, l = keys.length; i < l; i++) {
          let value = matched[i + 1];
          if (value) {
            params[keys[i]] = value;
          }
        }
        req.params = params;
        let action = route[1];
        action(req, res);
        return true;
      }
    }
    return false;
  };

  function handle(req, res) {
    let {pathname, query} = url.parse(req.url, true);
    req.query = query
    let method = req.method.toLowerCase();
    if (routes.hasOwnProperty(method)) {
      if (match(pathname, routes[method], req, res)) {
        return;
      } else {
        if (match(pathname, routes.all, req, res)) {
          return;
        }
      }
    } else {
      if (match(pathname, routes.all, req, res)) {
        return;
      }
    }
    handle404(req, res);
  }

  function handle404 (req, res) {
    res.end('404')
  }

  app.listen = function (...args) {
    const server = http.createServer((req, res) => {
      handle(req, res)
    })
    server.listen(...args)
  }
  return app
}

module.exports = express
```

其中 `express` 会把请求的方法都代理到 `app` 中作为属性的方式来方便用户使用。

```js
const express = require('./express')
const app = express()

app.use('/home/:id', (req, res) => {
  console.log(req.params)
  res.end('home')
})

// app.get 
// app.post 
app.get('/user', (req, res) => {
  console.log(req.query)
  res.end('user')
})

app.listen(8080, () => {
  console.log('port created successfully')
})
```

express 会在请求对象中加一些属性，会把路径参数作为请求时的 `params` 属性，会把查询字符串作为请求时的 `query` 属性。大多数中间件也是这个原理，如 body-parser 模块，给它加个 `body` 属性即可。

通过GitHub查看代码请点击：[传送门]()
