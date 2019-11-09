---
title: nginx配置防盗链
date: 2019-11-09 11:11:08
tags: Nginx
---
{% asset_img banner.png 图片 %}
<!-- more -->

防盗链的作用是禁止其他网站盗用自己网站资源。自己网站的资源只能自己使用。

{% asset_img http.png 图片 %}



首部字段 Referer 会告知服务器请求的原始资源的 URI。



客户端一般都会发送 Referer 首部字段给服务器。但当直接在浏览器的地址栏输入 URI，或出于安全性的考虑时，也可以不发送该首部字段。



在nginx中也是通过referer进行限制的，首先我们需要在`/etc/nginx/conf.d/default.conf`中进行如下配置。


```
location ~ .*\.(jpg|png|gif)$ {
　　　　valid_referers  none blocked 域名/IP;
　　　　if ($invalid_referer) {
　　　　　　return 403;
　　　　}
}
```


下面介绍字段的具体含义：



valid_referers  ：该指令会根据浏览器Referer header头的内容分配一个值0或1给变量$invalid_referer ;如果referer header 头不符合valid_referers指令设置的有效referer，变量$invalid_referer将被设置为1,那么便会执行if模块中的内容；



none ：默认值，表示没referer值的情况；blocked ：表示referer值被防火墙伪装；IP则是网站IP地址。



上面我是给图片加上防盗链了，所以其他用户的网站无法通过img外链到该图片资源。
