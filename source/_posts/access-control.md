---
title: nginx访问控制
date: 2019-11-09 11:25:38
tags: Nginx
---
{% asset_img banner.png 图片 %}

<!-- more -->

nginx常见的访问控制第一个基于ip的访问控制`http_access_module`模块，语法有两个。



allow语法
```
Syntax: allow [ address | CIDR | all ]
Default: —
Context: http, server, location, limit_except
```


deny语法
```
Syntax: deny [ address | CIDR | all ]
Default: —
Context: http, server, location, limit_except
```


名词解释
```
address  指IP地址(192.168.1.1)
CIDR     指网段(192.168.1.0/24)
all      所有人访问
```


禁止114.86.45.85访问,允许所有ip访问
```
location / {
   root   /opt/app/code;
   deny   114.86.45.85;
   allow  all;
   index  index.html index.htm;
}  
```


允许222.128.189.0/24网段ip访问，禁止所有ip访问
```
location ~ /{
   root   /opt/app/code;
   allow 222.128.189.0/24;
   deny all;
   index  index.html index.htm;
}
```


allow 和 deny是配合使用的。
