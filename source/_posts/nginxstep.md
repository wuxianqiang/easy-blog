---
title: 关于nginx的安装说明
date: 2019-09-28 21:24:41
tags: nginx
---
{% asset_img banner.png 图片 %}

之前已经了解到nginx适合做高性能的web服务器，那么接下来我将介绍nginx的安装和使用。

<!-- more -->

使用nginx之前我配置了一台centos7的操作系统，关于系统配置在之前文章已经介绍（这里是传送门）。


关于安装之前我们需要关闭防火墙
```
systemctl disable firewalld.service
```


停用selinux，通过`vi /etc/selnux/config`修改配置
```
SELLUNX=disabled
```


安全增强型 Linux 简称 SELinux，它是一个 Linux 内核模块，也是 Linux 的一个安全子系统。SELinux 主要由美国国家安全局开发。2.6 及以上版本的 Linux 内核都已经集成了 SELinux 模块。SELinux 的结构及配置非常复杂，而且有大量概念性的东西，要学精难度较大。很多 Linux 系统管理员嫌麻烦都把 SELinux 关闭了。



接下来我们需要安装依赖的模块
```
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
yum  -y install wget httpd-tools vim
```


使用yum安装需要添加安装源`vi /etc/yum.repos.d/nginx.repo`

```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
```


开始安装nginx
```
sudo yum install nginx
```


安装成功之后通过如下命令可以输出安装的版本
```
nginx -v
```

关于nginx的启动和关闭命令的使用

```
systemctl start nginx
systemctl stop nginx
```


在nginx中默认有两个配置，主配置文件地址
```
/etc/nginx/nginx.conf
```


主配置文件里面默认包含了一个子配置文件

```
/etc/nginx/conf.d/default.conf
```


不建议直接修改主配置文件，一般每个项目会自己建一个子配置文件，这样拆分的文件方便管理。
