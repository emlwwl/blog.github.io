---
title: SpringCloud Bus消息总线
date: 2021-05-23 20:05:55
tags: SpringCloud Bus
categories: 
- SpringCloud
---

> 官网：https://cloud.spring.io/spring-cloud-bus/reference/html/

## 一、概述

Spring Cloud Bus能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道。它整合了Java的事件处理机制和消息中间件的功能。
Spring Clud Bus目前支持RabbitMQ和Kafka。

Spring Cloud Bus 配合 Spring Cloud Config 使用可以实现配置的动态刷新。

<!--more-->

![img](https://springcloud-oss.oss-cn-shanghai.aliyuncs.com/chapter8/configbus1.jpg) 

根据此图我们可以看出利用Spring Cloud Bus做配置更新的步骤:

1. 提交代码触发post给客户端A发送bus/refresh
2. 客户端A接收到请求从Server端更新配置并且发送给Spring Cloud Bus
3. Spring Cloud bus接到消息并通知给其它客户端
4. 其它客户端接收到通知，请求Server端获取最新配置
5. 全部客户端均获取到最新的配置

## 二、配置及使用

首先必须先具备良好的RabbitMQ环境先，这里不做赘述。

#### 配置Config服务端

##### 1、pom.xml

```xml
<!--添加消息总线RabbitMQ支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

##### 2、application.yml

```
server:
  port: 3344

spring:
  application:
    name:  cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/willivie/springcloud-config.git #Gitee上面的git仓库名字
          ####搜索目录
          search-paths:
            - springcloud-config
      ####读取分支
      label: master

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

##rabbitmq相关配置,暴露bus刷新配置的端点
management:
  endpoints: #暴露bus刷新配置的端点
    web:
      exposure:
        include: 'bus-refresh'
```

#### 配置Config客户端

##### 1、pom.xml 和服务端一致

##### 2、bootstrap.yml

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取
      uri: http://localhost:3344 #配置中心地址
#rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"   # 'refresh'
```

配置后，修改Gitee上的配置文件内容，发送请求命令`curl -X POST "http://localhost:3355/actuator/bus-refresh"`，即可刷新所有客户端配置。

#### 刷新定点通知

不想全部通知，只想定点通知

公式：`http://localhost:配置中心的端口号/actuator/bus-refresh/{destination}`

示例：`"http://localhost:3344/actuator/bus-refresh/config-client:3355"`