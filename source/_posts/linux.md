---
title: Linux的基础知识
date: 2019-11-10 20:19:33
tags: Linux
---
{% asset_img banner.jpg 图片 %}

学习shell，通常会关联Linux，那么shell和Linux是什么关系了?

<!-- more -->

shell是一个具备特殊功能的程序。介于使用者和Linux操作系统之核心程序（kernel）间的一个接口。



为了对用户屏蔽内核的复杂性，也为了保护内核，以免用户误操作造成损害，在内核的周围建了一个外核-shell。






用户向shell提出请求，shell解释并将请求传给内核。

{% asset_img p1.png 图片 %}

## shell的版本



shell的分类

{% asset_img p2.png 图片 %}


(1) BourneShell(sh)：是由AT&T Bell实验室的 Steven Bourne为AT&T的Unix开发的，它是Unix的默认Shell，也是其它Shell的开发基础。Bourne Shell在编程方面相当优秀，但在处理与用户的交互方面不如其它几种Shell。



(2) BourneAgain Shell (即bash)：是自由软件基金会(GNU)开发的一个Shell，它是Linux系统中一个默认的Shell。Bash不但与Bourne Shell兼容，还继承了C Shell、Korn Shell等优点。



(3) CShell(csh)：是加州伯克利大学的Bill Joy为BSD Unix开发的，共有52个内部命令，与sh不同，它的语法与C语言很相似。它提供了Bourne Shell所不能处理的用户交互特征，如命令补全、命令别名、历史命令替换等。但是，C Shell与BourneShell并不兼容。该Shell其实是指向/bin/tcsh这样的一个Shell，也就是说，csh其实就是tcsh。



(4) KornShell(ksh)：是AT&T Bell实验室的David Korn开发的，共有42 条内部命令，它集合了C Shell和Bourne Shell的优点，并且与Bourne Shell向下完全兼容。Korn Shell的效率很高，其命令交互界面和编程交互界面都很好。



可以通过查看/etc/shells文件中的内容来查看当前主机中包含哪些类型的Shell，我们还可以通过在终端输入echo $SHELL 命令来查看当前使用的Shell类型。



## shell命令格式

第一个单词必须是命令的名字，后面空格隔开输入命令的选项或者参数。



{% asset_img p3.png 图片 %}


列出文件和文件夹的详细信息，ls是命令，-l是选项。



shell的高级操作管道操作功能

{% asset_img p4.png 图片 %}


命令1的输出是命令2的输入
```
ls -l | grep .ssh
```



在列出文件中查找.ssh文件信息
