---
title: koa源码分析
date: 2019-08-25 09:57:30
tags: Node
---
{% asset_img banner.png 图片 %}

<!-- more -->

使用步骤
```js
const Koa = require('koa');
const app = new Koa();
app.use(ctx => {
  ctx.body = 'Hello Koa';
});
app.listen(8080);
```

基本调用
```js
const http = require('http')

class Koa {
  constructor() {
    this.middleware = [];
  }
  listen (...args) {
    const server = http.createServer(this.callback());
    server.listen(...args)
  }
  use(fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
    this.middleware.push(fn);
    return this;
  }
  createContext (req, res) {
    // 请求对象和响应对象
    console.log(req, res)
  }
  callback () {
    const handleRequest = (req, res) => {
      const ctx = this.createContext(req, res)
    }
    return handleRequest
  }
}
```

处理context、request、response、req、res的引用关系
```js
const http = require('http')
const context = require('./context')
const request = require('./request')
const response = require('./response')

class Koa {
  constructor() {
    this.middleware = [];
    this.context = Object.create(context);
    this.request = Object.create(request);
    this.response = Object.create(response);
  }
  listen (...args) {
    const server = http.createServer(this.callback());
    server.listen(...args)
  }
  use(fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
    this.middleware.push(fn);
    return this;
  }
  createContext (req, res) {
    // request和response是封装的，req和res是原始的
    const context = Object.create(this.context);
    const request = context.request = Object.create(this.request);
    const response = context.response = Object.create(this.response);
    context.req = request.req = response.req = req;
    context.res = request.res = response.res = res;
    request.response = response;
    response.request = request;
  }
  callback () {
    const handleRequest = (req, res) => {
      const ctx = this.createContext(req, res)
    }
    return handleRequest
  }
}
```


request封装自定义的属性
```js
module.exports = {
  get url () {
    return this.req.url;
  },
  set url(val) {
    this.req.url = val;
  }
}
```

response封装自定义属性
```js
module.exports = {
  get body () {
    return this._body;
  },
  set body (val) {
    this._body = val;
  }
}
```

context代理request和response上的属性
```js
const delegate = require('./delegates');
const proto = module.exports = {}

delegate(proto, 'request').access('url')
```

通过getter和setter的形式代理
```js
module.exports = Delegator;

function Delegator(proto, target) {
  if (!(this instanceof Delegator)) return new Delegator(proto, target);
  this.proto = proto;
  this.target = target;
  this.getters = [];
  this.setters = [];
}

Delegator.prototype.access = function(name){
  return this.getter(name).setter(name);
};


Delegator.prototype.getter = function(name){
  var proto = this.proto;
  var target = this.target;
  this.getters.push(name);

  proto.__defineGetter__(name, function(){
    return this[target][name];
  });

  return this;
};

Delegator.prototype.setter = function(name){
  var proto = this.proto;
  var target = this.target;
  this.setters.push(name);

  proto.__defineSetter__(name, function(val){
    return this[target][name] = val;
  });

  return this;
};
```


通过use方法组合中间件
```js
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  return function (context, next) {
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

完整代码如下
```js
const http = require('http')
const context = require('./context')
const request = require('./request')
const response = require('./response')
const compose = require('./compose')

class Koa {
  constructor() {
    this.middleware = [];
    this.context = Object.create(context);
    this.request = Object.create(request);
    this.response = Object.create(response);
  }
  listen (...args) {
    const server = http.createServer(this.callback());
    server.listen(...args)
  }
  use(fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
    this.middleware.push(fn);
    return this;
  }
  createContext (req, res) {
    const context = Object.create(this.context);
    const request = context.request = Object.create(this.request);
    const response = context.response = Object.create(this.response);
    context.req = request.req = response.req = req;
    context.res = request.res = response.res = res;
    request.response = response;
    response.request = request;
  }
  callback () {
    const fn = compose(this.middleware);
    const handleRequest = (req, res) => {
      const ctx = this.createContext(req, res)
      return this.handleRequest(ctx, fn);
    }
    return handleRequest
  }
}
```
