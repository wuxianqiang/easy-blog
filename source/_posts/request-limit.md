---
title: nginx请求限制
date: 2019-11-09 11:14:19
tags: Nginx
---
{% asset_img banner.png 图片 %}
<!-- more -->

首先讲解两个算法：
{% asset_img request.jpg 图片 %}

令牌以固定速率产生，并缓存到令牌桶中；令牌桶放满时，多余的令牌被丢弃。


漏桶算法：
{% asset_img repose.png 图片 %}



水从上方倒入水桶，从水桶下方以固定速率流出；来不及流出的水存在水桶中，水桶满后水溢出。



Nginx按请求限制模块使用的是漏桶算法，即能够强行保证请求的实时处理速度不会超过设置的阈值。



官方限制ip并发连接和请求有两个模块，不需要重新编译安装，Nginx默认已经集成。


```
limit_req_zone  : 限制请求数
limit_conn_zone   ：限制并发连接数
```


limit_req_zone要和limit_req配合使用，配置目录是`/etc/nginx/conf.d/default.conf`


```
http {
   limit_req_zone  $binary_remote_addr  zone=one:10m   rate=1r/s;
   ...
     server { 
       ...
         location /search/ {
           limit_req   zone=one  burst=5 nodelay;
        }
```


1. $binary_remote_addr:远程的访问地址，此处以二进制的形式记录， 长度为 4 bytes
2. zone=one:10m :设置一个名字为one,大小为10M的缓存空间
3. rate=10r/s: 限制访问速率，此处设置为每秒接受10个请求
4. zone=one:指定使用名字为one的这个缓存空间，若没有设置burst参数，结合上文，此处的配置表示为每秒接受请求10个
5. burst=5:因为我们的流量并不是向漏桶一样每时每刻都是匀速的，所以为了避免某一时刻出现大规模的流量出现，所以我们添加burst参数，此处配置表示为，设置一个大小为5的缓冲区，当有大量请求（爆发）过来时，访问超过了上面的限制可以先放到缓冲区内。
6. nodelay:一般是和burst一起使用的，如果设置了nodelay，当访问超过了频次而且缓冲区也满的情况下会直接返回503，如果设置了，则所有大的请求会等待排队。





语法的书写规则
```
Syntax: limit_req_zone key zone=name:size rate=rate;   
Default: -- 
Context: http(定义在server以外)
```

```
Syntax: limit_req  zone=name [burst=number] [nodelay]; 
Default: -- 
Context: http,server,location
```


上面介绍的是请求上的限制，还有一个是连接上的限制，先要明白一点，一个TCP连接至少包含一个HTTP请求，所以也可以有多个HTTP请求。



语法书写的规则
```
Syntax: limit_conn_zone key zone=name:size;   
Default: -- 
Context: http(定义在server以外)
```

```
Syntax: limit_conn  zone number; 
Default: -- 
Context: http,server,location
```


使用起来也大同小异，具体配置如下
```
limit_conn_zone $binanry_remote_addr zone=conn_zone:1m; 
server {
  location /{
      limit_conn conn_zone 1;
  }
 }
 ```
