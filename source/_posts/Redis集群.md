---
title: Redis集群
date: 2021-03-20 11:05:55
tags: Redis
categories: 
- Redis
---

## 简单的主从复制，读写分离

### 1、 概述

是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master/leader)，后者称 为从节点(slave/follower)；数据的复制是单向的，只能由主节点到从节点。 Master以写为主，Slave 以读为主。

这样的好处是当集群中的某一个节点故障时，不会影响处理客户端请求的能力

<!--more-->

![](https://static01.imgkr.com/temp/e32a8e7956e94996ab26384202c03d36.png )

### 2、配置

启动时，服务器读取配置文件，并自动成为指定服务器的从服务器，从而构成主从复制的关系 。在主服务器开启IP绑定，在从服务器打开`redi.conf`文件，加入slaveof配置

- 主服务器配置

```
bind 127.0.0.1 192.168.1.8
protected-mode no
```

- 从服务器配置

```
slaveof 192.168.0.1 6379 
masterauth zn123
```

![](https://static01.imgkr.com/temp/aa885cad12a64bcb92acb1564861ab32.png )


将redis依次启动 ，先主后从

登录主redis客户端 ，看到connected_slaves:1就代表成功了

```
> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.1.8,port=6379,state=online,offset=13359,lag=1
master_repl_offset:13359
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:12030
repl_backlog_histlen:1330
```

向主服务器redis写入数据 

```
> set suceess true
OK
```

从服务器redis可读取数据

```
127.0.0.1:6379> get suceess
"true"
```

向从服务器写入数据失败，是因为从服务器默认配置slave-read-only yes，在redis.conf修改为no则可以向从Redis写数据

```
127.0.0.1:6379> set age 12 
(error) READONLY You can't write against a read only replica.
```
