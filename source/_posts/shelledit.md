---
title: shell基础入门
date: 2019-09-28 21:30:35
tags: Linux
---
{% asset_img banner.png 图片 %}

shell 是一个命令行解释器，它为用户提供了一个向Linux 内核发送请求以便运行程序的界面系统级程序。

<!-- more -->

脚本很多，shell也有很多种，通过以下命令查看
```
cat /ect/shells
```


编写 shell 脚本文件，新建文件 vi hello.sh，进入编辑时输入以下内容。
```
#!/bin/bash
echo hello world
```

按 ESC 退出编辑模式，通过 `:x` 保存，关于 vi 编辑器的使用在之前文章已经介绍了，（这里是传送门）


```
ls -ahl | grep hello.sh
```


通过详细的列出文件列表信息，可以查到文件的权限，文件的权限都是字母表示的。


```
-rw-r--r--.  1 root  root  29 Sep 19 13:21 hello.sh
```


第一位-表示文件，rw-表示所属者的权限，r--表示所属组的权限，再后面r--表示其他人的权限，默认创建的文件是没有执行权限的，我们需要修改权限。


```
chmod u+x hello.sh
```


通过给所属者增加执行的权限，那么就可以执行我们编写的shell脚本文件了。
```
./hello.sh
```

通过相对路径的方式执行脚本，可以在页面输出hello world的内容，这就是shell脚本的使用。
