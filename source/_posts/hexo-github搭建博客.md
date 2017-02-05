---
title: hexo-github搭建博客
date: 201７-02-04 19:10:05
tags: [npm, hexo, github]
categories: 教程
---

## 1.安装git
1.1 安装好git之后一定要配置好环境变量 *（PATH中添加X:\Git\bin;X:\Git\libexec\git-core）*否则提交代码到远程会报 git error
<!-- more -->
## 2.安装node.js
2.1 配置环境变量 *(貌似安装node.js时自动配置了，如果没有自行添加)*
2.2 在dos窗口执行node -v检查配置是否完成

## 3.配置github
3.1 如果没有github账号，则参考教程申请账号、配置连接 *[教程](http://note.youdao.com/noteshare?id=f9e40005274860eb585a07bb01f34416&sub=5BE205668CEE44BAB5B46113500A7D19)*
3.2 创建远程仓库blog用于管理博客源码,本地拉取blog的源码
3.3 创建远程仓库用于存放编译blog生成的public文件，也就是能够访问到的静态文件，命名格式为 ***github账号.github.io***

## 4.创建本地博客(执行命令用git bash)
4.1 安装hexo,执行命令
``` bash
npm install -g hexo-cli
```
4.2 新建一个localblog文件，执行命令生成hexo文件
``` bash
$ hexo init
```
<img src="/img/hexo/hexo-init.jpg" width=”240” height=”400”>

4.3 将localblog中生成的文件全部拷贝到远程拉取的blog项目中
<img src="/img/hexo/hexo-file.jpg" width=”240” height=”400”>

4.4 在blog目录下执行命令拉取依赖的包
``` bash
$ npm install
```
4.5 执行命令安装hexo-deployer-git
``` bash
$ npm install hexo-deployer-git --save
```
4.6 清除缓存
``` bash
$ hexo clean
```
4.7 编译
``` bash
$ hexo g
```
4.9 本地指定端口启动，localhost:5000访问
``` bash
$ hexo s -p 5000
```
## 5.部署到远程仓库***github账号.github.io***
5.1 修改配置文件_config.yml
``` bash
deploy:
  type: git
  repo:
    github: git@github.com:github账号/github账号.github.io.git
```
5.2 执行命令清除缓存 hexo clean
``` bash
$ hexo clean
```
5.3 编译
``` bash
$ hexo g
```
5.4 推送至远程仓库github账号.github.io
``` bash
$  hexo d
```
5.5 访问blog,https://github账号.github.io
