---
title: Redis安装
date: 2021-02-15 17:05:55
tags: Redis
categories: 
- Redis
---

## Linux下安装Redis
 ### 1、安装编译环境 
`yum install gcc-c++` 

### 2、下载redis安装包

链接：http://download.redis.io/releases/
下载后解压缩
`tar -xzvf redis-5.0.5.tar.gz  `

<!--more-->

![](https://static01.imgkr.com/temp/438306a6f02f467eab92459d77eb9bfb.png)

### 3、make 

这里可以直接make 是因为redis已经自己写好了make fifile 了,也就是说不用再执行confifigure 了
make 后编译好的文件会保存到src目录下 
` make & make install `

### 4、修改环境变量 

`export PATH=/usr/local/redis/bin:$PATH `

### 5、使用vi 修改redis.conf 

把daemonize no 变成daemonize yes 这样就可以让redis服务端后台启动

```
protected-mode no bind 0.0.0.0 
//后台 
daemonize yes
```

### 6、启动redis服务端 

` ./redis-server redis.conf `

检查一下是否启动成功，这里看的6379即代表启动成功

![](https://static01.imgkr.com/temp/91aa7feef9584fdb801e4371c3b9a502.png )

### 7、启动客户端 

```
redis-cli
redis-cli --raw #解决中文乱码问题
```

