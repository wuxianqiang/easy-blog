---
title: 快速了解nginx配置
date: 2019-09-28 08:15:21
tags: nginx
---
{% asset_img banner.png 图片 %}

学习nginx配置就像学习webpack配置一样，很多语法都是内部规定死的，我们只要按照规则配置即可。

<!-- more -->

在nginx的默认主配置文件`/etc/nginx/nginx.conf`
```
user  nginx;
worker_processes  1;
error_log /var/log/nginx / error.log warn;
pid /var/run/nginx.pid;
events {
  worker_connections  1024;
}
http {
  include / etc / nginx / mime.types;
  default_type  application / octet - stream;
  log_format  main  '$remote_addr;
  access_log /var/log/nginx / access.log  main;
  sendfile        on;
  #tcp_nopush     on;
  keepalive_timeout  65;
  #gzip  on;
  include / etc / nginx / conf.d/*.conf;
}
```


配置的格式是空格分开，然后使用分号结尾，#号开头的表示注释，http的表示关于http的配置，下面是关于里面关键字的说明。
```
user  管理使用用户
worker_processes  工作进程数量
error_log 指定错误日志目录
pid 进程号的保存目录
events {
  worker_connections  工作进程的最大连接数量
}
http {
  include 包含的文件类型
  default_type  默认的文件类型
  log_format  定义日志格式
  sendfile   零拷贝模式
  tcp_nopush  多个数据包合并
  keepalive_timeout  控制长连接时间
  gzip  是否启用压缩
  include 包含的其他配置文件
}
```


子配置文件在`/etc/nginx/conf.d/default.conf`
```
server {
  listen       80;
  server_name  localhost;
  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;
  location / {
      root   /usr/share/nginx/html;
      index  index.html index.htm;
  }
  #error_page  404              /404.html;
  error_page   500 502 503 504  /50x.html; 
  location = /50x.html {
      root   /usr/share/nginx/html;
  }
}
```


默认的http配置里面包含一个server配置，但是也可以配置多个，下面是具体的字段介绍。
```
server {
  listen      服务的端口号
  server_name  服务的名字
  #charset 服务的编码
  #access_log  日志文件目录
  location / { 
      root   访问/查找的静态文件目录
      index  默认的静态文件
  }
  #error_page  错误的状态码重定向的页面;
  error_page   500 502 503 504  /50x.html; 
  location = /50x.html {
      root   访问/50x.html访问的文件目录
  }
}
```


关于location的配置需要注意的是，location相当于路由，后面指定一个符号，关于符号大概有下面几种。

1. `=`   严格匹配。如果这个查询匹配，那么将停止搜索并立即处理此请求。   
2. `~`  为区分大小写匹配(可用正则表达式)    
3. `!~` 为区分大小写不匹配    
4. `~*` 为不区分大小写匹配(可用正则表达式)    
5. `!~*`为不区分大小写不匹配    
6. `^~` 如果把这个前缀用于一个常规字符串,那么告诉nginx 如果路径匹配那么不测试正则表达式。    

现在已经了解了nginx基本配置，下面是几个关键语法的特殊说明。

## sendfile配置

sendfile: 设置为on表示启动高效传输文件的模式。sendfile可以让Nginx在传输文件时直接在磁盘和tcp socket之间传输数据。如果这个参数不开启，会先在用户空间（Nginx进程空间）申请一个buffer，用read函数把数据从磁盘读到cache，再从cache读取到用户空间的buffer，再用write函数把数据从用户空间的buffer写入到内核的buffer，最后到tcp socket。开启这个参数后可以让数据不用经过用户buffer。

## tcp_nopush配置

一般情况下，在tcp交互的过程中，当应用程序接收到数据包后马上传送出去，不等待。也就是说tcp_nopush = on 结果就是数据包不会马上传送出去，等到数据包最大时，一次性的传输出去，这样有助于解决网络堵塞。这个配置和tcp_nodelay相反，表示立马传输。
