---
title: 你不知道的git高级技巧
date: 2019-11-03 09:40:35
tags: Git
---
{% asset_img banner.png 图片 %}

一年一度的双11活动都搞得沸沸扬扬，弄得满城风雨！
<!-- more -->



但是作为代码开发者，这个并不重要，因为只关心代码有没有bug，需求有没有提交。



说到代码提交，有些git命令还真不能忽视，往往却有着大用途，比如git cherry-pick命令。



可以选择某一个分支中的一个或几个commit(s)来进行操作。下面具体演示操作流程。



首先是我有两个分支master和dev，master有两次提交记录。


{% asset_img p1.png 图片 %}

dev分支是在第一次提交的时候切换出来的，而且没有第二次提交记录，第二次提交是在master上的，那么如何让master上的第二次提交转移到dev上呢?


{% asset_img p2.png 图片 %}

通过`git cherry-pick <commit id>`拉取其他分支提交记录，这样就防止了代码提交到了错误的分支。

{% asset_img p3.png 图片 %}


但是原来的分支还有错误的提交记录，也就是master还有第二次的提交记录，此时你可以放心的把第二次提交给删除了。

{% asset_img p4.png 图片 %}
