---
title: docker部署node项目
date: 2019-07-12 14:10:28
tags: Docker
---

{% asset_img banner.jpg 图片 %}

首先你要准备一台服务器，你可以选择购买各大厂商的服务器，你也可以选择自己搭建一个centos服务器，可以参考之前的文章了解[快速搭建centos7](https://mp.weixin.qq.com/s?__biz=MzI2ODY0ODM2Ng==&mid=2247485742&idx=1&sn=9717ffbb46931766b7774b8176fa8be8&chksm=eaed23efdd9aaaf9208a846978df04c361200f5a5fa3d5d809d6caebf73fa1fd03d0638ec8e7&token=1432388189&lang=zh_CN#rd)。

<!-- more -->

## Dockerfile

了解Dockerfile之前我们先介绍它是干什么的？

Dockerfile是一个配置文件，相当于一个node项目中的package.json文件，根据依赖来生成其他内容。

在使用docker前一定要先启动docker软件。

```
systemctl start docker
```

## 生成项目

这次我们选择的node项目是通过`express-generator`进行生成的。这个npm包可以生成一些简单的页面。

要使用`npm`我们需要先安装node，安装node要先安装nvm，安装流程如下。
```
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
nvm install stable
node -v
npm i cnpm -g
npm i nrm -g
```

使用`express-generator`也非常简单。命令如下，我们的项目文件夹存放在`app`目录。
```
npm install express-generator -g
express app
```

编写Dockerfile文件，自己通过命令建立一个这样的文件夹，如果不了解`vi`命令，请查看文章[vi编辑器](https://mp.weixin.qq.com/s?__biz=MzI2ODY0ODM2Ng==&mid=2247486094&idx=1&sn=ee2beeee81036e8155799de675058723&chksm=eaed204fdd9aa9599f61e914966a553ebc120d36e9edda6f8be75d3015c8e49bfa341efccbbc&token=932013802&lang=zh_CN#rd)。
```
vi Dockerfile
```
这个配置文件里面的内容
```
FROM node
COPY ./app /app
WORKDIR /app
RUN npm install
EXPOSE 3000
CMD npm start
```
参数`FROM`表示依赖的镜像，`COPY`表示将宿主机的文件拷贝到容器中，`WORKDIR`表示工作目录，`RUN`表示编译打包运行前执行的命令，`EXPOSE`表示容器暴露的断开，`CMD`表示容器中运行的命令。

了解更多参数访问[docker文档](http://www.dockerinfo.net/dockerfile%e4%bb%8b%e7%bb%8d)

编写Dockerfile完成之后，通过命令生成镜像
```
docker build -t express-demo .
```
`-t`表示指定镜像的名字，`.`表示在当前目录的`Dockerfile`开始构建。


然后就根据镜像运行我们的容器了。
```
docker container run -d -p 3333:3000 express-demo
```
通过IP:3333来访问当前的express项目吧。
