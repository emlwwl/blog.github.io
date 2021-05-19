---
title: Eureka注册中心
date: 2021-05-07 20:05:55
tags: Eureka
categories: 
- SpringCloud
---

> 中文版教程：https://www.springcloud.cc/spring-cloud-netflix.html

## 一、简介

**什么是Eureka？**

Eureka是springcloud中的一个负责**服务注册和发现**的组件。遵循着CPA理论中的A(可用性)，P(分区容错性)。

一个Eureka中分为**eureka server** 和**eureka client**。其中eureka是作为服务的注册和发现中心。eureka client 既可以作为服务的生产者，又可以作为服务的消费者。

**什么是服务注册与发现？**

<!--more-->

Eureka采用了CS的设计架构，Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka的客户端连接到 Eureka Server并维持**心跳连接**。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。
在服务注册与发现中，有一个**注册中心**。当服务器启动的时候，会把当前自己服务器的信息 （比如 **服务地址通讯地址**等）以**别名**方式注册到注册中心上。另一方（消费者/服务提供者），以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何rpc远程框架中，都会有一个注册中心(存放服务地址相关信息(接口地址))

## 二、部署Eureka Server

1、pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

2、application.yml

```bash
eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false

```

3、Main.java

```java
@SpringBootApplication
@EnableEurekaServer //代表是Eureka服务端
public class EurekaMain
{
    public static void main(String[] args)
    {
        SpringApplication.run(EurekaMain.class,args);
    }
}

```

4、启动项目测试，进入地址`http://localhost:7070`，看到下图即部署成功

![](https://static01.imgkr.com/temp/bb601bf4299e412ca1b13f362cf12776.png )

No instances available 代表没有服务被发现

## 三、部署Eureka Client

1、pom.xml

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

2、application.yml

```
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
  instance:
    instance-id: payment8001
    prefer-ip-address: true     #访问路径可以显示IP地址
```

3、Main.java

```
@SpringBootApplication
@EnableEurekaClient //代表是Eureka客户端
public class PaymentMain {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain.class,args);
    }
}
```

4、先启动Eureka Server项目，后启动Eureka Client。再次访问地址`http://localhost:7070` ，看到如图代表成功

![](https://static01.imgkr.com/temp/78b8d030f0204c678a9c72d81c6a8121.png )

> Eureka 健康检查：http://localhost:8001/actuator/health

