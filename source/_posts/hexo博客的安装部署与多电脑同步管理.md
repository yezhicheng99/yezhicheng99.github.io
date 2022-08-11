---
title: hexo博客的安装部署与多电脑同步管理
date: 2022-04-27 19:26:41
categories: hexo
tags: [hexo, 同步]
---

Hexo是一个非常方便好用的撰写博客的平台，但是有时候我们需要在不同电脑或者系统上撰写博客，这就会涉及到如何部署hexo在多台电脑上以及同步相关文档的问题。

## 在A电脑上部署 hexo

### 新建 hexo 文件夹并初始化博客
在A电脑的某个位置新建一个名为blog的文件夹，比如我的是 E:\blog

``` bash
$ cd /e/hexo/
$ hexo init
$ npm install hexo-deployer-git --save
```

### 新建github仓库
配置SSH Key，并将在本地生成的ssh key添加到github

在github上新建一个仓库名为：xxx.github.io的仓库，xxx为你的github用户名，如我的是：yezhicheng99.github.io

### 新建分支用于保存hexo博客源文件
``` bash
$ git init #git初始化
$ git remote add origin https://github.com/yezhicheng99/yezhicheng99.github.io.git #添加仓库地址
$ git checkout -b hexo #新建分支并切换到新建的分支
```

### 下载喜欢的主题并配置成 git submodule
这里我选的主题是：butterfly

``` bash
$ git submodule add https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```

手动打开blog文件夹下的 _config.yml 文件，找到 theme: 配置项并修改为
``` bash
theme: butterfly
```

### 修改配置文件

手动打开blog文件夹下的 _config.yml 文件，修改一下内容

``` bash
deploy:
  type: git
  repository: git@github.com:yezhicheng99/yezhicheng99.github.io.git
  branch: main
```

### 生成博客，并上传到github服务器
``` bash
$ hexo g # 生成博客
$ hexo s # 启动本地服务，这时候可以打开浏览器输入网址：http://localhost:4000/ 查看效果
$ hexo d # 上传到仓库main分支并部署到github服务器
```

这时候，在浏览器中输入： yezhicheng99.github.io 就可以看到自己写的博客了(这里可能会有延迟，如果没有更新一般等几分钟就可以了)

### 上传hexo源文件到github博客仓库下hexo分支
``` bash
$ git add . #添加所有本地文件到git
$ git commit -m "这里填写你本次提交的备注，内容随意" #git提交
$ git push origin hexo #文件推送到hexo分支
```


## 在B电脑上部署并同步hexo博客

### 下载github上hexo分支所有文件

``` bash
$ cd /workspace # cd到你想要保存博客文件的目录下
$ git clone --recursive -b hexo https://github.com/yezhicheng99/yezhicheng99.github.io.git blog　# clone所有模块，包括主题子模块
```

### 安装必要项
``` bash
$ npm install hexo
$ npm install hexo-deployer-git
$ npm install hexo-cli -g
$ npm install
```
如果报错没有关系，到这里你就可以书写新的博客了

### 书写新的博客并上传

``` bash
$ hexo new " 博客标题 "
```

打开 blog/source/_posts/ 目录下"博客标题.md"文件，并书写博客内容

``` bash
$ hexo clean
$ hexo g
$ hexo d
``` 

### 上传hexo源文件到github博客仓库下hexo分支
``` bash
$ git add . #添加所有本地文件到git
$ git commit -m " 这里填写你本次提交的备注，内容随意 " #git提交
$ git push origin hexo #文件推送到hexo分支
```

## 注意：每次在写新的博客之前，一定要先在blog目录下执行 git pull 或者 git pull origin hexo:hexo，获取最新源文件











