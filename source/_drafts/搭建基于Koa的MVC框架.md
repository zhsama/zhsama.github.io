---
title: 搭建基于Koa的MVC框架
date: 2021-10-26 01:20:01
updated: 2021-10-26 01:20:04
tags: ['koa', 'node', 'mvc']
categories: koa
---

# 一. 文章背景

Express 和 Koa 作为轻量级的的web框架，虽然项目配置灵活，几行代码即可启动后端服务，但随着业务复杂程度的提升，你很快就会发现需要自己手动配置各种  middleware，并且由于其轻量的特性，这些 web 框架并不约束项目的结构目录，因此不同水平的程序员搭建的框架质量也是参差不齐。为了解决这个问题，社区中涌现出基于Express 和 Koa 的 web 框架，比如 Egg.js 和 Nest.js。

最近摸鱼时间把Koa和公司框架源码简单看了一下，感觉收获巨大，于是决定手动实现一个简易且开箱即用的基于Koa的MVC框架。

该模板集成了Logger、Router、JWT、Mongoose、Redis，PM2等模块，还有部分的中间件集合。暂时没有考虑高并发处理，后期会继续完善。

本文的模块管理将采用 CommonJS 规范。

点击阅读前文前, 首页能看到的文章的简短描述

<!-- more -->

# 二. 目录结构

下面的目录是该框架的基础目录结构，后面的文章会对每一个目录进行详细介绍，让大家对项目的结构有更加清晰的了解。

```
├─bin                           // 启动目录
|  └www                         
├─controllers                   // 操作业务逻辑
|      ├─index.js               // 配置
|      ├─login.js               // 登录
|      └test.js                 // 测试
├─logs                          // 日志目录
|  ├─koa-template.log
|  └koa-template.log-2019-05-28
├─lib                           // 工具库
|  ├─error.js                   // 异常处理
|  └mongoDB.js                  // mongoDB配置
├─models                        // 数据库配置及模型
|   ├─index.js                  // 数据库配置
|   └user.js                    // 用户的schema文件
├─middlewares                   // 中间件
|      ├─cors.js                // 跨域中间件
|      ├─jwt.js                 // jwt中间件
|      ├─logger.js              // 日志打印中间件
|      └response.js             // 响应及异常处理中间件
├─routes                        // 路由
|   ├─private.js                // 校验接口
|   └public.js                  // 公开接口
├─services                      // 服务相关
|   ├─ proxy.js                 // 启动文件配置 
|      ├─index.js               // 配置
|      ├─user.js                // 用户
├─.gitignore                    // 忽略文件配置
├─app.js                        // 应用入口
├─config.js                     // 公共配置文件
├─ecosystem.config.js           // pm2配置文件
├─package.json                  // 依赖文件配置
├─README.md                     // README.md文档
```

# 三. 搭建过程

## 1. 搭建环境

因为 node.js v7.6.0 开始完全支持 async/await ，所以 node.js 环境都要 7.6.0 以上。本文的node环境为 14.17.6，nvm版本为 6.14.15。

1. 初始化 package.json
    * 执行`npm init -y`
2. 安装 koa2
    * 执行`npm install -s koa`

执行1、2步骤后文件中会下图文件，但是这肯定不是我们需要的，那么接下就来开始搭建基本框架结构。

![image-20211026014443204](C:\Users\QAQ\AppData\Roaming\Typora\typora-user-images\image-20211026014443204.png)

## 2. 新建services文件夹

### 1. 新建启动文件 proxy.js

新建 proxy 文件，用于部署的时候可以启动整个后端程序，也就是前端中的集成的运行环境。后端的运行、关闭、重启都在这文件进行即可。基本代码如下：

```js
#!/usr/bin/env node

/**
 * Module dependencies.
 */

const app = require('../app')
const http = require('http')
const config = require('../config')

/**
 * Get port from environment and store in Express.
 */

const port = normalizePort(process.env.PORT || config.port)
// app.set('port', port);

/**
 * Create HTTP server.
 */

const server = http.createServer(app.callback())

/**
 * Listen on provided port, on all network interfaces.
 */

server.listen(port)
server.on('error', onError)
server.on('listening', onListening)

/**
 * Normalize a port into a number, string, or false.
 */

function normalizePort(val) {
  const port = parseInt(val, 10)

  if (isNaN(port)) {
    // named pipe
    return val
  }

  if (port >= 0) {
    // port number
    return port
  }

  return false
}

/**
 * Event listener for HTTP server "error" event.
 */

function onError(error) {
  if (error.syscall !== 'listen') {
    throw error
  }

  const bind = typeof port === 'string'
    ? 'Pipe ' + port
    : 'Port ' + port

  // handle specific listen errors with friendly messages
  switch (error.code) {
    case 'EACCES':
      console.error(bind + ' requires elevated privileges')
      process.exit(1)
      break
    case 'EADDRINUSE':
      console.error(bind + ' is already in use')
      process.exit(1)
      break
    default:
      throw error
  }
}

/**
 * Event listener for HTTP server "listening" event.
 */

function onListening() {
  const addr = server.address()
  const bind = typeof addr === 'string'
    ? 'pipe ' + addr
    : 'port ' + addr.port
  console.log('Listening on ' + bind)
}
```

相信用过 koa-generator 的朋友对这个代码并不陌生，这其实就是 generator 的代码，express项目的启动文件也基本差不多。它的基本思路就是利用了 node.js 中的 http 模块，让 http 暴露端口并进行监听，这个端口是在配置文件 config.js 中引入的。
