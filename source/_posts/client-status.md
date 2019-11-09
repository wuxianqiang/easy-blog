---
title: 监控nginx客户端的状态
date: 2019-11-09 11:23:01
tags: Nginx
---
{% asset_img banner.png 图片 %}

<!-- more -->

Nginx由内核和模块组成，其中，内核的设计非常微小和简洁，完成的工作也非常简单，仅仅通过查找配置文件将客户端请求映射到一个location block（location是Nginx配置中的一个指令，用于URL匹配），而在这个location中所配置的每个指令将会启动不同的模块去完成相应的工作。


使用模块 `--with-http_stub_status_module` 监控nginx客户端的状态。语法如下
```
Syntax: stub_status on/off;
Default: - 
Context: server->location
```


使用需要配置 `/etc/nginx/conf.d/default.conf`
```
server {
    location /status {
    stub_status  on;
}
```



保存配置之后重启 Nginx 服务
```
systemctl reload nginx.service
```


重启完成根据自己电脑IP访问 `/status` 目录
```
http://192.168.0.165/status
```


页面显示如下内容说明配置成功
{% asset_img status.png 图片 %}
