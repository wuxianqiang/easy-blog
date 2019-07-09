---
title: shell中的变量
date: 2019-07-04 16:40:33
tags: Linux
---
{% asset_img banner.png 图片 %}

什么是变量？

1. 可以变化的量
2. 变量必须以字母或下划线开头，名字中间只能由字母，数字和下划线组成
3. 变量名的长度不得超过255个字符
4. 变量名在有效范围内必须唯一
5. 变量默认类型都是字符串

<!-- more -->

演示一下定义变量和引用变量的过程
```
[root@localhost ~]# name=张三
[root@localhost ~]# echo $name
张三
[root@localhost ~]# echo "$name"11
张三11
[root@localhost ~]# echo "${name}"11
张三11
[root@localhost ~]# 
```

## 两种变量

1. 环境变量是全局变量，而自定义变量是局部变量
2. 自定义变量会在当前的shell中生效，而环境变量会在当前shell以及其子shell中生效

通过`env`查看环境变量，`export 变量名=变量值` 这样设置环境变量，`$PATH` 查看环境变量
```
[root@localhost myshell]# echo $PATH
/root/.nvm/versions/node/v11.14.0/bin:/usr/local/sbin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/java/jdk1.8.0_211/bin:/root/bin
```

如果想把一个自定义的脚本直接可以执行`hello.sh`，或者把这个文件拷贝到目标目录下`/root/bin`，或者把脚本所在目录添加到环境变量中的PATH路径中

默认是不可以直接在命令行输入`hello.sh`的，只有添加之后才可以执行。
```
// /root/myshell
#!/bin/bash
echo hello
```
记得要把文件权限修改成可执行的权限。
```
export PATH="$PATH":/root/myshell   临时生效
hello.sh
```
预定义变量：是脚本中已经定义好的变量，`$$` 当前进程的进程号(PID)

## read
从键盘读取变量值，`read [选项] [变量名]`
```
// 选项
-p：指定读取值时的提示符；
-t：指定读取值时等待的时间（秒）。
```
案例演示
```
#!/bin/bash
read -p '请输入一个数字' -t 5 name
echo "您输入的数字是$name"
```
