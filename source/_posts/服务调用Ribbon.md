---
title: 服务调用Ribbon
date: 2021-05-15 11:05:55
tags: Ribbon
categories: 
- SpringCloud
---
> 官网：https://github.com/Netflix/ribbon

> 中文文档：http://docs.springcloud.cn/user-guide/ribbon/

## 一、简介

### 什么是Ribbon？

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**客户端负载均衡**的工具。

Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法和服务调用。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。

### LB负载均衡(Load Balance)是什么？

简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）。
常见的负载均衡有软件Nginx，LVS，硬件 F5等。

<!--more-->

### Ribbon本地负载均衡客户端 VS Nginx服务端负载均衡区别？

 Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即负载均衡是由服务端实现的。

 Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。

##  二、Ribbon的七种负载均衡策略

| 名称                      | 解释                                                         |
| ------------------------- | ------------------------------------------------------------ |
| RoundRobinRule            | 轮询策略                                                     |
| RandomRule                | 随机策略                                                     |
| BestAvailableRule         | 过滤出故障服务器后，选择一个并发量最小的                     |
| WeightedResponseTimeRule  | 针对响应时间加权轮询                                         |
| AvailabilityFilteringRule | 可用过滤策略，先过滤出故障的或并发请求大于阈值的一部分服务实例，然后再以线性轮询的方式从过滤后的实例清单中选出一个; |
| ZoneAvoidanceRule         | 从最佳区域实例集合中选择一个最优性能的服务实例               |
| RetryRule                 | 选择一个Server，如果失败，重新选择一个Server重试             |

## 三、一起使用Ribbon和Eureka

### 1、pom.xml引入Eureka时，包含了Ribbon的依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

![](https://static01.imgkr.com/temp/04884a4201544954b85887fabcecbb94.png )

### 2、配置RestTemplate

```java
@Configuration
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced //使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### 3、服务消费方

```java

@RestController
public class OrderController {

    public static final String PaymentSrv_URL = "http://CLOUD-PAYMENT-SERVICE";

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult getPayment(@PathVariable Long id) {
        return restTemplate.getForObject(PaymentSrv_URL + "/payment/get/" + id, CommonResult.class, id);
    }
}
```

### 4、调用时，服务提供方的端口号交替出现则代表负载均衡配置成功

## 四、项目中更换负载均衡策略

### 1、配置细节

官方文档明确给出了警告：
这个自定义配置类不能放在@ComponentScan所扫描的当前包（@SpringBootApplication注解下的主启动类所在的包）下以及子包下，
否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了。

### 2、新增自定义规则类 实现负载均衡随机策略

```java
@Configuration
public class MySelfRule
{
    @Bean
    public IRule myRule()
    {
        return new RandomRule();//定义为随机
    }
}
```

### 3、主启动类添加注解

```java
@SpringBootApplication
@EnableEurekaClient
//在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效，
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration=MySelfRule.class)
public class OrderMain80
{
    public static void main(String[] args)
    {
        SpringApplication.run(OrderMain80.class,args);
    }
}

```

