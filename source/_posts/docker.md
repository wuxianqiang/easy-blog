---
title: 开始上手docker
date: 2019-07-09 08:01:15
tags: Docker
---
{% asset_img banner.png 图片 %}

每个软件都有自己依赖的环境，docker是Linux容器的封装。

这次我们是在centos7操作系统中使用docker软件。

<!-- more -->

关于怎么安装centos7操作系统的介绍请看[快速搭建centos7
](https://mp.weixin.qq.com/s?__biz=MzI2ODY0ODM2Ng==&mid=2247485742&idx=1&sn=9717ffbb46931766b7774b8176fa8be8&chksm=eaed23efdd9aaaf9208a846978df04c361200f5a5fa3d5d809d6caebf73fa1fd03d0638ec8e7&token=966055462&lang=zh_CN#rd)。



## 安装docker

```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --enable docker-ce-test
yum install docker-ce docker-ce-cli containerd.io
```

关于具体的安装详细说明请参照[官方文档](https://docs.docker.com/install/linux/docker-ce/centos/)


## 快速上手

下面是使用docker的流程图说明：

{% asset_img linux.jpg 图片 %}

1. `docker pull`，先从远程拉取镜像
2. 拉到本地镜像仓库 `image` 文件夹
3. `docker run` 通过镜像生成容器

首先我们要明白一点的是镜像和容器的关系，通过类和实例来比喻，镜像就相当于一个类，而容器就相当于类的实例，我们可以通过镜像生成很多个容器。

## 常用命令

在操作系统中启动docker软件。
```
systemctl start docker
```

所有demo都是从hello-world写起，我们也例外，首先，我们执行拉取镜像命令。
```
docker pull hello-world
```
拉取的镜像我们通过以下命令查看。
```
docker image ls
```
然后通过镜像生成我们的容器。
```
docker run hello-world
```
容器里面的脚本运行完成将会在控制台打印如图所示的信息。

{% asset_img log.png 图片 %}

前台没有了进程工作，容器就会退出，但是容器文件还操作系统中，查看我们的容器。

```
docker container ls -a
```
查看到的信息里面包含容器ID，容器状态等。


`-a`表示查看所有的容器，不传表示查看正在运行的容器。

容器一直保留在计算机中很占用资源，我们要删除掉。
```js
docker rm 容器ID
```

如果本地不想保留这个镜像，我们也可以把镜像删除掉。
```
docker rmi hello-world
```

### 使用nginx镜像

通常我们会使用nginx作为静态服务器，因为它的性能强悍，处理并发非常合适。

我们先拉取镜像
```
docker pull nginx
```
通过镜像生成我们的容器
```
docker run --name port_nginx -p 8080:80 nginx
```
`--name`表示指定容器的名称，没有指定也会自动随机生成，`-p 8080:80`表示指定端口，宿主机的8080端口指向容器里面的80端口。

`ip addr show`来查看我们的IP，通过IP:8080用浏览器访问。我们可以看到如下所示的页面。

{% asset_img nginx.png 图片 %}

然后控制台每次访问都会输出日志，可以按`ctrl+c`暂停，服务就会停止。

{% asset_img log2.png 图片 %}

这就是docker的日常使用，你都学会了吗?
