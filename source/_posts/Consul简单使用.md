---
title: Consul简单使用
date: 2021-05-13 20:05:55
tags: Consul
categories: 
- SpringCloud
---

> 中文文档：https://www.springcloud.cc/spring-cloud-consul.html

> 下载地址：https://www.consul.io/downloads

## 一、简介

Consul 是一套开源的分布式服务发现和配置管理系统，由 HashiCorp 公司用 Go 语言开发。

提供了微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。

它具有很多优点。包括： 基于 raft 协议；比较简洁；支持健康检查；同时支持 **HTTP** 和 **DNS** 协议 ；支持跨数据中心的 **WAN 集群** ；提供**图形界面** ；跨平台，支持 Linux、Mac、Windows

<!--more-->

## 二、运行Consul

下载后只有一个.exe文件，双击即可启动

安装成功后，查看版本号 `consul --version`

```
E:\consul_1.9.5_windows_amd64>consul --version
Consul v1.9.5
Revision 3c1c22679
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
```

用开发者模式启动 `consul agent -dev`

```
E:\consul_1.9.5_windows_amd64>consul agent -dev
==> Starting Consul agent...
           Version: '1.9.5'
           Node ID: '714bd79e-253d-46ae-f785-c28173dd0bf6'
         Node name: 'DESKTOP-PM0NM4F'
        Datacenter: 'dc1' (Segment: '<all>')
            Server: true (Bootstrap: false)
       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, DNS: 8600)
      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false, Auto-Encrypt-TLS: false

==> Log data will now stream in as it occurs:
.......
```

打开`http://localhost:8500` 看到下图代表启动成功

![](https://static01.imgkr.com/temp/93ee014f4a73427f99a427a6af74502d.png) 

## 三、部署到项目

1、pom.xml

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

2、application.yml

```bash
## consul服务端口号
server:
  port: 8006

spring:
  application:
    name: consul-provider-payment
  ## consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}
```

3、Main.java

```java
@SpringBootApplication
@EnableDiscoveryClient   //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
public class PaymentMain {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8004.class,args);
    }
}
```

4、启动项目，进入地址`http://localhost:8500`，看到下图即服务注册成功

![](https://static01.imgkr.com/temp/5d5b04ca542c4b1db5eb54ab8ccf716c.png )

