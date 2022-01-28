---
title: 利用 Github Actions 自动部署 Hexo 博客
date: 2022-01-28 17:35:57
updated: 2022-01-28 17:35:57
tags: ['CI','Github Actions','Hexo']
categories: 
  - CI
  - Hexo
description: 不摆烂第二期！花了三四个小时终于折腾好 Github Actions 自动部署 Hexo 了，以后拖稿的理由又少了一个！（Travis 全面收费了，不得不转平台，罪大恶极！）
---

[toc]

# 介绍

Github Actions 可以很方便实现 CI/CD 工作流，类似 Travis 的用法，来帮我们完成一些工作，比如实现自动化测试、打包、部署等操作。当我们运行 Jobs 时，它会创建一个容器 (runner)，容器支持：Ubuntu、Windows 和 MacOS 等系统，在容器中我们可以安装软件，利用安装的软件帮我们处理一些数据，然后把处理好的数据推送到某个地方。在 Travis 全面收费之后，笔者不得不换一个 CI 平台来完成自动化部署，便有了这篇文章。

本文将介绍利用 Github Actions 实现自动部署 hexo 到 Github Pages。在之前我们需要写完文章后在本地执行 `hexo g -d` 来部署，当你文章比较多的时候，编译部署需要等待很长时间。而且还可能会遇到本地安装的 Node.js 版本与 Hexo 不兼容的问题，需要用 `nvm` 来管理不同版本的Node.js（其实主要还是因为懒）。。利用 Github Actions 你将会没有这些烦恼，仅需要把Hexo项目pull到本地，把本地写好的文章放入项目的 `_posts` 文件夹下然后 push，Actions将自动部署更新。

# 前提

## 创建所需仓库

本文仅用一个仓库存储静态文件和hexo项目，分不同的branch存储：

1. 创建 `blog` 分支用来存放 Hexo 项目
2. 创建 `main` 分支用来存放静态博客页面

## 生成部署密钥

一路按回车直到生成成功

```
$ ssh-keygen -f github-deploy-key
```

当前目录下会有 `github-deploy-key` 和 `github-deploy-key.pub` 两个文件。

![image](https://user-images.githubusercontent.com/33454514/151518040-6204df2e-413e-40c0-bb9f-2dbcf61b35f2.png)

## 配置部署密钥

复制 `github-deploy-key` 文件内容，在 `blog` 仓库 `Settings -> Secrets -> Actions -> Add a new secret` 页面上添加。

1. 在 `Name` 输入框填写 `HEXO_DEPLOY_PRI`。
2. 在 `Value` 输入框填写 `github-deploy-key` 文件内容（以`-----BEGIN OPENSSH PRIVATE KEY-----`开头）。

![image](https://user-images.githubusercontent.com/33454514/151518361-36101e8d-db94-4362-b7f7-6e9497ce0880.png)

复制 `github-deploy-key.pub` 文件内容，在 `your.github.io` 仓库 `Settings -> Deploy keys -> Add deploy key` 页面上添加。

1. 在 `Title` 输入框填写 `HEXO_DEPLOY_PUB`。
2. 在 `Key` 输入框填写 `github-deploy-key.pub` 文件内容。![image](https://user-images.githubusercontent.com/33454514/151518605-f89c8566-d231-4cdc-9cd7-ca6cad0eecd4.png)
3. 勾选 `Allow write access` 选项。![image](https://user-images.githubusercontent.com/33454514/151518731-37a7c402-a78f-40b0-a25a-9f8d742fa053.png)

# 编写 Github Actions

## Workflow 模版

在仓库的 `blog` 分支下根目录下创建 `.github/workflows/deploy.yml` 文件，目录结构如下。

```
blog
└─ .github
    └─ workflows
        └─ deploy.yml
```

在 `deploy.yml` 文件中粘贴以下内容。

```yml
name: Hexo
on:
  push:
    branches:
      - blog
env:
  GIT_USER: yourname
  GIT_EMAIL: yourname@example.com
  DEPLOY_REPO: yourname/yourname.github.io
  DEPLOY_BRANCH: main

jobs:
  build:
    name: Build on node ${{ matrix.node_version }} and ${{ matrix.os }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        os: [ ubuntu-latest ]
        node_version: [ 14.x ]

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Checkout deploy repo
        uses: actions/checkout@v2
        with:
          repository: ${{ env.DEPLOY_REPO }}
          ref: ${{ env.DEPLOY_BRANCH }}
          path: .deploy_git

      - name: Use Node.js ${{ matrix.node_version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node_version }}

      - name: Configuration environment
        env:
          HEXO_DEPLOY_PRI: ${{ secrets.HEXO_BUILD_PRI }}
        run: |
          sudo timedatectl set-timezone "Asia/Shanghai"
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 700 ~/.ssh
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name $GIT_USER
          git config --global user.email $GIT_EMAIL

      - name: Install dependencies
        run: |
          npm install hexo-cli -g
          npm install

      - name: Deploy hexo
        run: |
          hexo clean
          hexo deploy
```

## 模版参数说明

- `name`为此 Action 的名字
- `on`触发条件，当满足条件时会触发此任务，这里的 `on.push.branches.blog` 是指当 `blog` 分支收到 `push` 后执行任务。
- env为环境变量对象
  - `env.GIT_USER` 为 Hexo 编译后使用此 git 用户部署到仓库。
  - `env.GIT_EMAIL` 为 Hexo 编译后使用此 git 邮箱部署到仓库。
  - `env.DEPLOY_REPO` 为 Hexo 编译后要部署的仓库，例如：`zhsama/zhsama.github.io`。
  - `env.DEPLOY_BRANCH` 为 Hexo 编译后要部署到的分支，例如：master。
- jobs为此 Action 下的任务列表
  - `jobs.{job}.name` 任务名称
  - `jobs.{job}.runs-on` 任务所需容器，可选值：`ubuntu-latest`、`windows-latest`、`macos-latest`。
  - `jobs.{job}.strategy` 策略下可以写 `array` 格式，此 job 会遍历此数组执行。
  - `jobs.{job}.steps`一个步骤数组，可以把所要干的事分步骤放到这里。
    - `jobs.{job}.steps.$.name` 步骤名，编译时会会以 LOG 形式输出。
    - `jobs.{job}.steps.$.uses` 所要调用的 Action，可以到 https://github.com/actions 查看更多。
    - `jobs.{job}.steps.$.with` 一个对象，调用 Action 传的参数，具体可以查看所使用 Action 的说明。

## 配置文件

最终目录结构

```
blog
└─ .github
    └─ workflows
        └─ deploy.yml
```

把 `deploy.yml` 文件推送到 `blog` 分支，在此仓库 `Actions` 页面可以看到一个名字为 `HEXO` 的 Action。

## 执行任务

写一篇文章，`push` 到仓库的 `blog` 分支，在此仓库 `Actions` 页面查看当前的task。

![image](https://user-images.githubusercontent.com/33454514/151520844-e97f8697-3501-4edf-9d05-56271db059ef.png)

当任务完成后查看您的博客 `https://your.github.io`，如果不出意外的话已经可以看到新添加的文章了。

# 小结

偷懒是人类发展的最大动力！！！还是微软爸爸有钱，Actions免费用，这样我的守望先锋2是不是未来可期了啊？🤤🤤🤤

