---
title: 删除中间的git提交
date: 2019-11-03 09:37:56
tags: Git
---
{% asset_img banner.png 图片 %}

在这买买买的年代里！！！这年头谁的购物车还没几件想买的东西呀，昨天老大还跟我说他把定金都付了呢，哈哈！

<!-- more -->




不扯了，现在开始讲解最近学的git命令，你们搬好小板凳，我要开始了！！！



往往git提交有很多commit记录，但是如果因为中间失误存在错误提交，此时你需要撤销中间的代码，我们需要使用git revert命令。



比如现在我的git有三次提交记录

{% asset_img p1.png 图片 %}


现在想要把中间的第二次提交给删除掉，保留其他两次。使用`git revert <commit id>`操作。

{% asset_img p2.png 图片 %}


如果代码有冲突，需要自己解决冲突，修改完成代码之后需要 `git add .` 和 `git revert --continue` 合并代码。

{% asset_img p3.png 图片 %}


最后你代码已经删除掉了第二次提交，完成了操作。git 历史有撤销的记录。

{% asset_img p4.png 图片 %}
