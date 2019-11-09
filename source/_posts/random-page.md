---
title: nginx配置随机主页
date: 2019-11-09 11:20:14
tags: Nginx
---
{% asset_img banner.png 图片 %}

Nginx由内核和模块组成，其中，内核的设计非常微小和简洁，完成的工作也非常简单，仅仅通过查找配置文件将客户端请求映射到一个location block（location是Nginx配置中的一个指令，用于URL匹配），而在这个location中所配置的每个指令将会启动不同的模块去完成相应的工作。

<!-- more -->

`--with-http_random_index_module` 在根目录里随机选择一个主页显示
```
Syntax: random_index on/off; 
Default: off 
Context: location
```


使用需要配置 `/etc/nginx/conf.d/default.conf`
```
location / {
       root /opt/app;
       random_index on;
}
```

当前配置的是 `/opt/app`，里面需要创建 html 文件，如1.html,2.html,3.html，在访问的时候会随机选择一个html文件进行显示



保存配置之后重启 Nginx 服务
```
systemctl reload nginx.service
```


重启完成根据自己电脑IP访问
```
http://192.168.0.165
```


访问的页面就是 nginx 在 `/opt/app` 目录里面随机选择的内容
