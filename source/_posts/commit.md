---
title: 制作个性化镜像
date: 2019-07-09 22:47:03
tags: Docker
---
{% asset_img banner.png 图片 %}

我们可以制作我们自己的镜像来生成容器去运行。

制作镜像有commit和Dockerfile两种形式。

<!-- more -->

## commit方式

commit的方式就是一种类似于git的形式，先改文件，改完文件再提交就行了，详细步骤是是这样的。

我们以nignx镜像为例，在镜像里面添加点东西，就成了我们自己的镜像。

首先是从远程的镜像仓库拉取我们要用的nginx镜像，这个过程就相当于我们安装npm包一样，下载我们需要的文件到本地。

```
docker pull nginx
```

然后通过镜像生成我们的容器
```
docker run -it --name nginx1 nginx bash
```
由于我们需要进入到容器里面通过终端进行操作，所有传入`-it`参数，`i`是命令交换式的意思，`t`是分配终端的意思。`--name`表示生成容器的名字，`bash`表示运行的脚本。

分配了终端之后我们可以在终端输入一些命令，比如创建文件操作呀
```
touch hello-world.txt
```

添加完我想要的文件之后，我们输入`exit`退出容器，此时容器是停止的，没有再运行了。

最后就是提交镜像
```js
docker container commit -m "我制作的镜像" -a "my-name" nginx1 my-nginx:v1
```
参数`-m`和git提交的`-m`一个意思，表示文字说明，`-a`表示作者，`nginx1`表示的是哪个容器，`my-nginx:v1`表示镜像的名字和标签。

镜像添加完成我们可以同命令查看是否添加正确了。
```
docker image ls
```
```
REPOSITORY   TAG   IMAGE ID            CREATED             SIZE
my-nginx     v1     0358f985a53d     7 seconds ago     109MB
```
这就是一个制作个性化镜像的过程，其实更多的时候会选择Dockerfile的方式制作镜像，这个我们下次文章分享。
