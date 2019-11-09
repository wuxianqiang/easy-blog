---
title: 使用Nginx作为HTTP负载均衡
date: 2019-11-09 11:07:47
tags: Nginx
---
{% asset_img banner.png 图片 %}

跨多个应用服务器的负载均衡是一种用于优化资源利用率，最大化吞吐量，减少延迟和确保容错配置的常用技术。

<!-- more -->

可以将nginx用作非常有效的HTTP负载均衡，以将流量分配到多个应用服务器，并使用nginx改善Web应用程序的性能，可伸缩性和可靠性。



nginx支持以下负载均衡机制（或方法）：
1. 轮询-对应用服务器的请求以轮询方式分发
2. 最少连接-将下一个请求分配给活动连接数量最少的服务器
3. ip-hash-哈希函数用于确定应为下一个请求选择哪个服务器（基于客户端的IP地址）






## 默认负载均衡配置



使用nginx进行负载均衡的最简单配置如下所示：
```
http {
   upstream myapp1 {
       server srv1.example.com;
       server srv2.example.com;
       server srv3.example.com;
   }

   server {
       listen 80;

       location / {
           proxy_pass http://myapp1;
       }
   }
}
```


在上面的示例中，一个nginx服务在三个应用服务器srv1-srv3上运行。如果未特别配置负载均衡机制，则默认为循环。所有请求都被代理到服务器组myapp1，nginx应用HTTP负载均衡来分发请求。



其中nginx中的反向代理实现包括HTTP，HTTPS，FastCGI，uwsgi，SCGI，memcached和gRPC的负载均衡。



要为HTTPS（而非HTTP）配置负载均衡，只需使用“ https”作为协议。


## 最少连接的负载均衡



另一个负载均衡原则是连接最少的。最少连接允许在某些请求需要较长时间才能完成的情况下更公平地控制应用服务器的负载。


使用最少连接的负载均衡，nginx将尝试不使繁忙的应用服务器因过多的请求而过载，而是将新的请求分配给不太繁忙的服务器。


当minimum_conn指令用作服务器组配置的一部分时，将激活nginx中的最低连接负载均衡：
```
upstream myapp1 {
       least_conn;
       server srv1.example.com;
       server srv2.example.com;
       server srv3.example.com;
   }
```

## 会话持久性



请注意，通过循环或最少连接的负载均衡，每个后续客户端的请求都可能分配到其他服务器。不能保证将同一客户端始终定向到同一服务器。 



如果需要将客户端绑定到特定的应用程序服务器（换句话说，就始终尝试选择特定服务器而言，使客户端的会话“粘滞”或“持久”），可以使用ip-hash负载均衡机制。



使用ip-hash，客户端的IP地址用作哈希密钥，以确定应为客户端的请求选择服务器组中的哪个服务器。此方法可确保将来自同一客户端的请求始终定向到同一服务器，除非该服务器不可用。


要配置ip-hash负载均衡，只需将ip_hash指令添加到服务器（上游）组配置中：

```
upstream myapp1 {
   ip_hash;
   server srv1.example.com;
   server srv2.example.com;
   server srv3.example.com;
}
```

## 加权负载均衡



还可以通过使用服务器权重来进一步影响nginx负载均衡算法。 



在上面的示例中，未配置服务器权重，这意味着所有指定的服务器都被视为对特定负载均衡方法具有同等资格。 



特别是对于循环，这也意味着在服务器之间的请求分配大致相等-前提是有足够的请求，并且以统一的方式处理请求并足够快地完成请求。 



当为服务器指定权重参数时，权重将作为负载均衡决策的一部分。


```
upstream myapp1 {
       server srv1.example.com weight=3;
       server srv2.example.com;
       server srv3.example.com;
   }
```


使用此配置，每5个新请求将按以下方式分布在应用程序实例中：3个请求将定向到srv1，一个请求将定向到srv2，另一个将定向到srv3。



类似地，在最新版本的nginx中，可以使用权重在最少的连接和ip-hash负载均衡。
