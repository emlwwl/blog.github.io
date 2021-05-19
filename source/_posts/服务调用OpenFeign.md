---
title: 服务调用OpenFeign
date: 2021-05-16 11:05:55
tags: OpenFeign
categories: 
- Spring Cloud
---

> Github：https://github.com/spring-cloud/spring-cloud-openfeign

## 一、简介

### 什么是Feign?

Feign是一个声明式的Web服务客户端，让编写Web服务客户端变得非常容易，只需**创建一个接口并在接口上添加注解即可**

### Feign的作用？

Feign旨在使编写Java Http客户端变得更容易。
前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

<!--more-->

### Feign集成了Ribbon

利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，通过feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用

## 二、Feign和OpenFeign的区别？

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign是Spring Cloud组件中的一个轻量级RESTful的HTTP服务客户端。Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务 | OpenFeign是Spring Cloud 在Feign的基础上支持了SpringMVC的注解，如@RequesMapping等等。OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |
| 依赖：spring-cloud-starter-feign                             | 依赖：spring-cloud-starter-openfeign                         |

## 三、项目中集成OpenFeign

1、pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2、yml

```bash
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```

3、主启动

```java
@SpringBootApplication
@EnableFeignClients
public class OrderFeignMain
{
    public static void main(String[] args)
    {
        SpringApplication.run(OrderFeignMain80.class,args);
    }
}
```

4、新增Service 与要调用的服务器提供方路径和方法名保持一致

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService
{
    @GetMapping(value = "/payment/get/{id}")
    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
}
```

## 四、OpenFeign的超时控制

使用@FeignClient调用接口默认等待1秒钟，但是服务端处理若是超过1秒钟，导致Feign客户端不想等待了，直接返回报错。

```java
Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is feign.RetryableException: connect timed out executing GET http://CLOUD-PAYMENT-SERVICE/payment/feign/timeout] with root cause
java.net.SocketTimeoutException: connect timed out
	at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method) ~[na:1.8.0_201]
```

为了避免这样的情况，有时候我们需要设置Feign客户端的超时控制。yml文件中开启配置

```xml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/

#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
#指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
#指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

## 五、OpenFeign日志功能

Feign 提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解 Feign 中 Http 请求的细节。
说白了就是对Feign接口的调用情况进行监控和输出

**日志级别**

- NONE：默认的，不显示任何日志；
- BASIC：仅记录请求方法、URL、响应状态码及执行时间；
- HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；
- FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据。

**集成到项目**

1、配置Bean

```java
@Configuration
public class FeignLog {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

2、yml

```bash
#设置feign客户端超时时间
#springCloud默认开启支持ribbon
ribbon:
#指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
#指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000

logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.wwl.springcloud.service.PaymentFeignService: debug
```

3、后台查看日志

```java
2021-05-16 21:11:13.905 DEBUG 16712 --- [p-nio-80-exec-1] c.w.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] <--- HTTP/1.1 200 (2320ms)
2021-05-16 21:11:13.905 DEBUG 16712 --- [p-nio-80-exec-1] c.w.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] connection: keep-alive
2021-05-16 21:11:13.905 DEBUG 16712 --- [p-nio-80-exec-1] c.w.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] content-type: application/json
2021-05-16 21:11:13.905 DEBUG 16712 --- [p-nio-80-exec-1] c.w.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] date: Sun, 16 May 2021 13:11:13 GMT
2021-05-16 21:11:13.905 DEBUG 16712 --- [p-nio-80-exec-1] c.w.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] keep-alive: timeout=60
2021-05-16 21:11:13.905 DEBUG 16712 --- [p-nio-80-exec-1] c.w.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] transfer-encoding: chunked
2021-05-16 21:11:13.905 DEBUG 16712 --- [p-nio-80-exec-1] c.w.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] 
2021-05-16 21:11:13.907 DEBUG 16712 --- [p-nio-80-exec-1] c.w.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] {"code":200,"message":"查询成功\t 服务端口：8001","data":{"id":1,"serial":"hkjhkj"}}
2021-05-16 21:11:13.907 DEBUG 16712 --- [p-nio-80-exec-1] c.w.s.service.PaymentFeignService        : [PaymentFeignService#getPaymentById] <--- END HTTP (93-byte body)
```

