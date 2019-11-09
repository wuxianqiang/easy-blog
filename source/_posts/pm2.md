---
title: pm2部署
date: 2019-11-09 09:49:00
tags: Node
---

{% asset_img banner.png 图片 %}


pm2 是一个带有负载均衡功能的 Node 应用的进程管理器。 它使您可以使应用程序永远保持活动状态，无需停机即可重新加载它们，并简化常见的系统管理任务。

<!-- more -->

这里的PM2，不是PM2.5，跟空气没有半毛钱的关系。它是Node应用的进程管理器，可以利用它来简化很多Node应用管理的繁琐任务，如性能监控、自动重启、负载均衡等。



之前我们启动一个项目通过node app.js这样启动，现在可以通过pm2 start app.js启动，前提是你必须通过npm 安装了pm2模块。



今天主要介绍通过node启动一个静态资源服务器，然后部署到远程服务器中，部署的是Vue生成的项目。


```
vue create vue-pm2
```


通过Vue的脚手架去生成Vue的项目文件，然后再通过命令生成pm2的配置文件。


```
pm2 ecosystem
```

此时我项目文件目录是这样的

{% asset_img file.png 图片 %}

默认的pm2配置是这样的
```js
module.exports = {
  apps : [{
    name: 'API',
    script: 'app.js',
    // Options reference: https://pm2.io/doc/en/runtime/reference/ecosystem-file/
    args: 'one two',
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production'
    }
  }],

  deploy : {
    production : {
      user : 'node',
      host : '212.83.163.1',
      ref  : 'origin/master',
      repo : 'git@github.com:repo.git',
      path : '/var/www/production',
      'post-deploy' : 'npm install && pm2 reload ecosystem.config.js --env production'
    }
  }
};
```

post-deploy 中最后指定 --env production 表示我们会用到 apps 中 env_production 的配置。区分部署的环境，通常会部署开发环境和生成环境两种。

```js
module.exports = {
  /**
   * Application configuration section
   * http://pm2.keymetrics.io/docs/usage/application-declaration/
   */
  apps : [
    // First application
    {
      name      : 'API',
      script    : 'app.js',
      append_env_to_name: true,
      env: {
        COMMON_VARIABLE: 'true'
      },
      env_production : {
        NODE_ENV: 'production'
      }
    }
  ],

  /**
   * Deployment section
   * http://pm2.keymetrics.io/docs/usage/deployment/
   */

  deploy : {
    production : {
      user : 'root',
      host : '192.168.4.90',
      ref  : 'origin/master',
      repo : 'git@github.com:wuxianqiang/vue-pm2.git',
      path : '/var/www/production',
      'post-deploy' : 'git pull && npm install && npm run build && pm2 reload ecosystem.config.js --env production'
    },
    dev : {
      user : 'root',
      host : '192.168.4.90',
      ref  : 'origin/master',
      repo : 'git@github.com:wuxianqiang/vue-pm2.git',
      path : '/var/www/development',
      'post-deploy' : 'git pull && npm install && npm run build && pm2 reload ecosystem.config.js --env dev',
      env  : {
        NODE_ENV: 'dev'
      }
    }
  }
};
```
pm2的配置完成了，现在需要使用koa搭建一个静态资源服务器。

```js
// app.js
const Koa = require('koa');    
const path = require('path');    
const koaStatic = require('koa-static')    
const app = new Koa();    
const NODE_ENV = process.env.NODE_ENV || 'dev'    
const portMap = {    
  production: 4000,    
  dev: 3000    
}    
const root = path.resolve(__dirname, 'dist')    
app.use(koaStatic(root));    
app.listen(portMap[NODE_ENV]);    
```
需要在package.json中添加运行的脚本命令
```
 "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint",
    "deploy:production": "pm2 deploy ecosystem.config.js production",
    "deploy:dev": "pm2 deploy ecosystem.config.js dev",
    "setup:production": "pm2 deploy ecosystem.config.js production setup",
    "setup:dev": "pm2 deploy ecosystem.config.js dev setup"
  },
```
在运行命令之前你需要把本机的git公钥配置到远程服务器中，还需要把远程服务器的公钥配置到GitHub中，这样才能保证代码可以正常拉取。



现在开始运行脚本，部署开发环境代码，第一条命令只在第一次部署的时候才要运行，之后再次部署不要运行命令了。
```
npm run setup:dev
npm run deploy:dev
```


部署生成环境代码
```
npm run setup:production
npm run deploy:production
```
