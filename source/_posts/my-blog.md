---
title: 第一个博客
date: 2020-11-10 23:29:53
tags:
- 零
categories: 
- 生活
---

## 收获

>记载通过卢克的视频学习到的东西，视频链接https://www.bilibili.com/video/BV1dt4y1Q7UE

1. hexo博客搭建
2. github静态网站部署
3. 通过github的Action自动部署项目

## 介绍

| 相关说明\名称 | Hexo       | VuePress   | Gatsby        |
| ------------- | ---------- | ---------- | ------------- |
| 难易度        | 1          | 2          | 3             |
| 技术栈        | ejs/stylus | vue/stylus | react/graphQL |
| 主题丰富度    | 3          | 1          | 2             |
| 插件          | 3          | 1          | 3             |
| 功能性        | 1          | 2          | 3             |
| github stars  | 30.8k      | 16.9k      | 45.6k         |

hexo:https://hexo.bootcss.com/
vuepress:https://www.vuepress.cn/
gatsby:https://www.gatsbyjs.org/

## 使用建议

- 新手   Hexo
- Vue   VuePress
- React   Gatsby
- 自己造轮子~

## 功能

1. 基本功能
2. github pages
3. 自动化部署
4. 在线编辑

## 开始

> tips：初始化超慢... 建议翻墙~

```
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

## 部署

整体的目的就是把项目提交到git上后在做配置
github有两种方式

- 方式一：https://[username].github.io 仓库名必须是[username].github.io 打包产物master分支 一般用于主页
ps:比如我的用户名是SixMango，那么我创建的仓库名叫 sixmango.github.io (不区分大小写)，因为这样才会把该项目作为二级域名下的根，不明白先操作就知道啦~

- 方式二：https://[username].github.io/[repo] 可以自定义仓库名称， 一般用于小项目demo

**命令行自动化部署**

1. 创建仓库 - 创建github上对应的仓库
2. 添加依赖 - yarn add hexo-deployer-git   添加deply模块化与git做关联
3. 修改配置 - _config.yml的deploy文件
4. 执行命令  npm/yarn run deploy  把生成的解压public文件夹下的文件会全量覆盖目标库~ 

```yml
deploy:
    type: git
    repo: git@github.com:SixMango/sixmango.github.io.git
    branch: main
```
上面已经可以完成通过命令完成博客更新了

**github的Action更新后自动部署**

下面完成的是通过github的Action完成自动化部署，就是提交代码后自动发布，类似webhook

1. 创建一个my-blog的分支
2. 创建 .github\workflows\deploy.yml  ps：windows手动创建带点的文件会出现特殊字符创建不了  用cmd命令窗口切换到当前目录  运行 mkdir .github\workflows\
3. 创建deploy.yml文件  内容如下  意思是说  检查，运行什么命令，怎么部署，api文档在github的每个项目里面action选项里面有
4. 提交代码，后  修改下文章后提交my-blog分支，去gitlib上看actions的运行是否正常执行

```yml
name: Build and Deploy
on: [push]
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
        name: Checkout 🛎️
        uses: actions/checkout@v2 # If you're using actions/checkout@v2 you must set persist-credentials to false in most cases for the deployment to work correctly.
        with:
          persist-credentials: false

        name: Install and Build 🔧 # This example project is built using npm and outputs the result to the 'build' folder. Replace with the commands required to build your project, or remove this step entirely if your site is pre-built.
        run: |
          npm install
          npm run build
        env:
          CI: false
    
        name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@releases/v3
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          BRANCH: main # The branch the action should deploy to.
          FOLDER: public # The folder the action should deploy.
```

## 在线编辑

通过修改ejs的模板加一个编辑的超链接
ps：利用github自己的编辑功能

```xml
<div>
  <a target="_blank" href="你的项目编辑页面链接不要后缀<%- page.source %>">编辑/a>
</div>
```