---
title: Github的几种加速下载方式
date: 2020-09-113 20:05:55
tags: 博客
categories: 
- 博客
---

### 1\. GitHub 镜像访问

这里提供两个最常用的镜像地址：

* https://github.com.cnpmjs.org
* https://hub.fastgit.org

也就是说上面的镜像就是一个克隆版的 GitHub，你可以访问上面的镜像网站，网站的内容跟 GitHub 是完整同步的镜像，然后在这个网站里面进行下载克隆等操作。

<!--more-->

### 2\. GitHub 文件加速

利用 Cloudflare Workers 对 github release 、archive 以及项目文件进行加速，部署无需服务器且自带CDN.

* https://gh.api.99988866.xyz
* https://g.ioiox.com

以上网站为演示站点，如无法打开可以查看开源项目：gh\-proxy\-GitHub\(https://hunsh.net/archives/23/\) 文件加速自行部署。

### 3.Github 加速下载

只需要复制当前 GitHub 地址粘贴到输入框中就可以代理加速下载！

地址：http://toolwa.com/github/



### 4\. 加速你的 Github

https://github.zhlh6.cn

输入 Github 仓库地址，使用生成的地址进行 git ssh 等操作


### 5\. 谷歌浏览器 GitHub 加速插件\(推荐\)

![](https://s3.ax1x.com/2020/11/18/Dmj9Ve.png)


### 6\. GitHub raw 加速

GitHub raw 域名并非 github.com 而是 raw.githubusercontent.com，上方的 GitHub 加速如果不能加速这个域名，那么可以使用 Static CDN 提供的反代服务。

将 raw.githubusercontent.com 替换为 raw.staticdn.net 即可加速。

### 7\. GitHub \+ Jsdelivr

jsdelivr 唯一美中不足的就是它不能获取 exe 文件以及 Release 处附加的 exe 和 dmg 文件。

也就是说如果 exe 文件是附加在 Release 处但是没有在 code 里面的话是无法获取的。所以只能当作静态文件 cdn 用途，而不能作为 Release 加速下载的用途。







