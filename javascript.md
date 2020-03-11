---
title: javascript
date: 2088-08-08 08:08:08
tags:
---

# javascript
```
#JavaScript介绍
JavaScript 是脚本语言
JavaScript 是一种轻量级的编程语言。
JavaScript 是可插入 HTML 页面的编程代码。
JavaScript 插入 HTML 页面后，可由所有的现代浏览器执行。
JavaScript 很容易学习。
```

# jquery
```
#jQuery介绍
jQuery是一个快速、简洁的JavaScript框架，是继Prototype之后又一个优秀的JavaScript代码库（或JavaScript框架）

#使用
<head>
    <script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
</head>
```

#nodejs&npm
``` bash
#nodejs介绍
简单的说 Node.js 就是运行在服务端的 JavaScript。
Node.js 是一个基于Chrome JavaScript 运行时建立的一个平台。
Node.js是一个事件驱动I/O服务端JavaScript环境，基于Google的V8引擎，V8引擎执行Javascript的速度非常快，性能非常好。
#npm介绍
NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题

#nodejs&npm安装
nodejs官网下载最新稳定版本

#卸载
sudo rm -rf /usr/local/{bin/{node,npm},lib/node_modules/npm,lib/node,share/man/*/node.*}

#npm使用
npm list -g depth 0
##npm全局安装vue全家桶
npm install -g  @vue/cli
##项目局部安装
npm install --registry=https://registry.npm.taobao.org
```

#vue
```bash
#vue介绍
Vue是一套用于构建用户界面的渐进式JavaScript框架。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，方便与第三方库或既有项目整合。

#安装命令行工具
npm install -g @vue/cli

#常用命令
##打印vue-cli工具版本  vue版本主要以package.json 或 <script>中引用的js，都应为2.X版本
vue -V

##查看运行环境
vue info

##vue2.0从模版初始化项目，webpackage为vue模版，也可引用任何其他github中的模版
vue init webpackage my-project

##vue3.0初始化项目, 不推荐使用 vue3.0基本已废弃
vue create my-project


##vue模版
webpack
vue-admin-template

##vue UI
element ui  饿了吗
```
