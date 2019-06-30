---
title: 自动部署你的博客
date: 2019-06-01 10:18:50
tags: Share
---
{% asset_img blog.jpg 图片 %}

通过 Hexo 快速生成静态博客，通过 GitHub 代码托管，然后通过 CI 网站持续部署。

<!-- more -->

# 生成博客

Hexo 是一个快速、简洁且高效的博客框架。接下来只需要使用 npm 即可完成 Hexo 的安装。
```
npm install -g hexo-cli
```

执行命令之前请确保你已经安装了 node 和 git 软件。

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```
hexo init <folder>
cd <folder>
npm install
```

`<folder>` 表示项目名称，可以自定义，然后 `cd` 表示进入到生成的项目，`npm install` 表示安装依赖。

# 上传代码

你需要有一个 GitHub 账号登录，然后新建一个空仓库，把刚才生成的项目提交的这个仓库中。

生成 GitHub 的 token，详细步骤如下图。
{% asset_img opensetting.png 图片 %}
{% asset_img createtoken.png 图片 %}

选中所有的选项
{% asset_img settingtoken.png 图片 %}

复制生成的token
{% asset_img copytoken.png 图片 %}

# 自动部署

`https://travis-ci.com/` 通过 GitHub 登录到这个自动部署的网站中，这个网站会自动获取你 GitHub 中的所有仓库，选择你刚才创建的项目，修改设置选项。

增加环境变量，名字是 TOKEN，值是通过 GitHub 生成的 `token`，是登录 GitHub 的令牌。
{% asset_img inputtoken.png 图片 %}

# 添加配置

修改博客的配置文件，将部署的GitHub地址和部署的跟目录加上。保证静态资源可以正常访问。
{% asset_img changeroot.png 图片 %}
{% asset_img changeaddress.png 图片 %}

增加 `.travis.yml` 文件，里面的内容如下，需要改成你的用户名邮箱和 git 项目地址。
```
language: node_js
node_js:
  - "11"
install:
  - npm install
script:
  - hexo g
after_script:
  - cd ./public
  - git init
  - git config user.name "用户名"
  - git config user.email "邮箱"
  - git add -A
  - git commit -m "init"
  - git push --force "https://${TOKEN}@项目地址" "master:gh-pages"
branches:
  only:
    - master
```

完成以上操作，然后再推送一次到 GitHub 中，项目就会自动开始部署到 GitHub 的 `gh-pages` 中，每次推送都会自动部署的，关于项目的地址就是的静态页地址。格式是：`用户名.gitbub.io/项目名称`。
