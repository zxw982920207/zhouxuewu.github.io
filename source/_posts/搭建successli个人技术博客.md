---
title: 搭建successli个人技术博客

date: 2017-7-20

tags: hexo
---

![](http://pccmxww5q.bkt.clouddn.com/GitHub+HEXO.png?imageView2/0/w/560/h/380/q/100 "")

<!-- more -->


`successli`技术博客搭建采用[Hexo](https://hexo.io/zh-cn/index.html)，一款快速、简洁且高效的博客框架。下面是具体的搭建和更新博客的步骤。

## 环境准备

1. 根据 [官网文档](https://hexo.io/zh-cn/docs/index.html) 安装`NodeJS`和`Git`。
2.  安装`NodeJS`完成以后，使用`npm`安装`hexo-cli`博客管理工具，如果网络不好这个过程通常很慢。

```shell
$ npm install -g hexo-cli
```

## 创建博客

如果你是博客的发布人员，直接查看[**发布文章**](#发布文章)段落。
1. 创建`Github`仓库`successli.github.io`
2. 初始化博客系统

```shell
hexo init successli.github.io
```

3. 进入博客目录，并初始化博客。

```shell
cd successli.github.io/
npm install
```

4. 修改博客基本信息  
在`_config.yml`里面修改标题和描述

5. 配置博客插件。  
当前项目目录安装发布工具，

```sh
npm install hexo-deployer-git --save
```

6. 同时安装启动服务插件，以便本地可以启动
```sh
npm install hexo-server --save
```
7. 然后在`_config.yml`里面配置如下内容：
```yml
deploy: 
  type: git
  repo: https://github.com/successli/successli.github.io
  branch: master
```
8. 关联`Github`仓库，并把源码推送到远程，因为`master`是留给生成文件的，所以发布到了非`master`分支。
```sh
git init
git remote add origin https://github.com/successli/successli.github.io.git
git checkout -b source
git add .
git commit -m "init blog"
git push -u origin source
```
9. 发布博客
直接运行如下命令发布博客，该命令会自动发布内容到`master`分支
```sh
hexo deploy
```

## 发布文章
1. 如果本地没有仓库请`clone`仓库，并且切换到`source`分支。
```
git clone https://github.com/successli/successli.github.io.git
git branch source
```
2. `Setup`本地环境
在项目目录运行如下命令安装依赖和初始化环境。
```sh
npm install
```
3. 运行如下命令创建博客文章，后面的参数便是文章的标题
```sh 
hexo new '搭建successli个人技术博客'
```
4. 在`source/_posts`目录找到刚才对应的文章，进入编辑文档即可。编辑过程中可以使用如下命令启动服务器和实时预览效果。
```
hexo server
hexo generate --watch
```
5. 发布源码 
编辑文章完成以后运行如下命令发布源码到`Github`仓库
```
git add .
git commit -m "add new post"
git push origin source
```
6. 发布文章
```
hexo generate --deploy
```


参考： [搭建DailyCast技术博客](https://dailycast.github.io/搭建DailyCast技术博客/)