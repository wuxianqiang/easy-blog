---
title: 使用svrx来mock数据
date: 2019-12-08 16:39:04
tags: Mock
---

{% asset_img banner.png 图片 %}

svrx(server-x) 是一个渐进且易于使用的、插件化的前端开发工作台。官方链接：https://docs.svrx.io/zh/

<!-- more -->

有了svrx可以帮我们做静态私服，接口代理，文件改变时自动刷新页面，还有动态路由功能等等。



使用要从安装开始
```
 npm install -g @svrx/cli
```


哪个文件夹需要作为静态资源的文件夹就在哪里执行命令svrx即可，即可启动服务给我们访问。

{% asset_img demo.png 图片 %}


动态路由功能可以帮助我们mock接口数据，通过访问指定的URL然后返回已经定义好的数据。首先需要一个路由的配置文件，我们新建route.js文件，然后通过命令启动服务器。
```
svrx --route route.js
```


在 route.js 中配置如下代码
```js
get('/blog').to.json({ title: 'svrx' });
```


打开 `/blog`，你将看到 `{title: 'svrx'}` 的 json 输出在页面中显示出来了。



关于路由的配置文件的语法规则是这样的。
```
[method](selector).to.[action](payload)
```

路由方法method属性和express框架类似，路由的匹配规则selector属性也和express框架类似。



关于acion属性支持send发送内容
```js
get('/blog').to.send({ title: 'this is a blog' });
```

而且支持sendFile发送文件
```js
get('/index.html').to.sendFile('./index.html');
```


而且路由还可以重定向的
```js
get('/blog').to.redirect('/user');
```


接口代理的配置也是很简单
```js
get('/api(.*)').to.proxy('http://mock.server.com/')
```


往往在前后端开发过程中，我们可以使用svrx来模拟接口返回的数据，配置也简单易学，提高我们的工作效率也带来良好的开发体验。
