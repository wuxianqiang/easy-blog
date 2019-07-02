---
title: 在Node环境中使用mongodb
date: 2019-06-30 22:09:39
tags: Node
---

{% asset_img banner.jpg 图片 %}

MongoDB是一个基于分布式文件存储的数据库

<!-- more -->

## MongoDB安装

使用数据库之前先安装mongodb服务端，官方下载：[传送门](https://www.mongodb.com/download-center#community)

1. 下载之后存放到 `C:\Program Files\MongoDB`
{% asset_img mg1.png 图片 %}

2. 在除C盘外的盘符新建一个空目录，如 `D:\Mongodb\data`
3. 执行命令 `mongod --dbpath=D:\Mongodb\data` 挂载数据盘

{% asset_img mg2.png 图片 %}

1. 如果出现 `waiting for connections on port 27017` 就表示启动成功,已经在27017端口上监听了客户端的请求
2. 注意：`--dbpath` 后的值表示数据库文件的存储路径,而且后面的路径必须事先创建好，必须已经存在，否则服务开启失败
3. 注意：这个命令窗体绝对不能关,关闭这个窗口就相当于停止了mongodb服务

## mongoose安装

MongoDB服务端已经启动，在Node环境中要用JS语法操作数据库，要使用mongoose包，相当于操作数据库的客户端。

```js
// 在node环境中使用mongoose
const mongoose = require('mongoose')

const {Schame, model} = mongoose
// mongodb 协议
// localhost 本地IP
// databasename 数据库名称，没有会自动创建
mongoose.connect('mongodb://localhost/databasename', {
  useNewUrlParser: true
})

//  连接是异步的，通过回调通知连接成功
const db = mongoose.connection
db.once('open', () => {
  console.log('数据连接成功')
})
db.once('error', () => {
  console.log('数据连接失败')
})
```

定义表单的结构，并且在模型中使用它
```js
const { Schema, model } = require('mongoose')

const userSchema = new Schema({
  email: String,
  password: String,
  salt: String,
  username: String
})

module.exports = model('user', userSchema)
```
实现数据的增，删，改，查
```js
const db = require('./user')
db.create({
  email: '123@qq.com',
  password: '123',
  salt: '123',
  username: '123'
})
db.deleteOne({ username: '123' }
db.updateOne({ username: '123' }, { username: '456' })
db.findOne({ username: '123' }
```