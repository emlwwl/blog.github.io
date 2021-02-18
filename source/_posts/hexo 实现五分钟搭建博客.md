
---
title: hexo 实现五分钟搭建博客
date: 2020-09-13 20:05:55
tags: Hexo
categories: 
- 博客
---


Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，
在几秒内，即可利用靓丽的主题生成静态网页。
我也是一时心血来潮想搭建一个博客玩玩，也就网上查了下教程，花了几天搭建好了，欢迎来访：[](http://localhost:4000/)[Mr\.Wu’s Blog](http://emlwwl.github.io/)

## 具体实现步骤：

1. 安装Git bash
2. 安装Node.js
3. 安装Hexo
4. 在Github创建个人仓库
5. 将博客部署到Github
6. 发布文章
<!--more-->


关于一二步不多做赘述，百度的教程一大堆，下载后安装即可。

### [](https://emlwwl.github.io/2020/05/11/hexo%20实现五分钟搭建博客/#安装Hexo)安装Hexo

安装好Node.js后，打开命令行窗口，执行以下命令：



安装后可用hexo -v查看版本：



接下来创建自己的博客目录（在任意位置创建一个myblog文件夹即可）

在myblog文件夹打开命令行窗口，并输出命令：

首先初始化博客
```hexo init```

接下来安装依赖
```npm install```


执行完毕后，myblog目录下后出现以下文件：

* node\_modules: 依赖包
* public：存放生成的页面
* scaffolds：生成文章的一些模板
* source：用来存放你的文章
* themes：主题

### [](https://emlwwl.github.io/2020/05/11/hexo%20实现五分钟搭建博客/#Github创建个人仓库)Github创建个人仓库

首先得注册github，注册后创建仓库（详细图文可百度查找），找到new repository




![](https://emlwwl.github.io/images/eg1.png)


创建一个和你用户名相同的仓库，后面加上github.io，例如：username.github.io ，只有这样，将来要部署到GitHub page的时候，才会被识别，其中username就是你注册GitHub的用户名。我这里是已经建过了。

### [](https://emlwwl.github.io/2020/05/11/hexo%20实现五分钟搭建博客/#将hexo部署到github)将hexo部署到github

首先全局配置你的Git用户名和邮箱，详情可看教程：[Git基本操作](https://emlwwl.github.io/2020/05/05/Git基本操作/)

配置后将生成的ssh密钥添加到github，如图：

[](https://emlwwl.github.io/images/eg2.png)

查看ssh是否配置成功：
```ssh -T git@github.com```


然后就可以开始部署到Github，在myblog目录下打开命令行窗口，顺序执行下列命令

```
npm install hexo-deployer-git --save  #安装自动部署发布工具

hexo clean  #清除缓存

hexo generate  #保存修改，生成文件

hexo deploy  #发布到远程
```
完成后即可访问username.github.io看到你的博客啦！

[](https://emlwwl.github.io/images/eg3.png)
看到这张图，就代表部署成功啦

### [](https://emlwwl.github.io/2020/05/11/hexo%20实现五分钟搭建博客/#发布文章)发布文章

部署成功后就可以开始写博文了，文章是.md格式，文件目录默认创建在/myblog/source/\_posts下

新建文章命令：
```hexo new 'xxx文件名'```


执行后可在本地运行查看效果
```hexo g && hexo s```


最后推荐一个快速简洁好用的markdown编辑器→Typroa，百度搜索下载即可












