---
title: Gateway服务网关
date: 2021-05-21 20:05:55
tags: Gateway
categories: 
- SpringCloud
---

> 官网：https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/

## 一、概述

**简介**

Gateway是在Spring生态系统之上构建的API网关服务，基于Spring 5，Spring Boot 2和 Project Reactor等技术。
Gateway旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能， 例如：熔断、限流、重试等

SpringCloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 1.x非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

<!--more-->

**相关概念**

- Route（路由）：这是网关的基本构建块。它由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配。
- Predicate（断言）：这是一个 Java 8 的 Predicate。输入类型是一个 ServerWebExchange。我们可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。
- Filter（过滤器）：这是`org.springframework.cloud.gateway.filter.GatewayFilter`的实例，我们可以使用它修改请求和响应。

**工作流程**

![](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/images/spring_cloud_gateway_diagram.png)

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。

Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。

Filter在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，
在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。

## 二、配置与使用

### 1、Route（路由）

#### Spring Cloud Gateway 网关路由有两种配置方式：

- 在配置文件 yml 中配置
- 通过`@Bean`自定义 RouteLocator，或者在在启动主类 Application 中配置 

#### pom.xml 引入依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

#### 方式一：application.yml

```bash
server:
  port: 8080
spring:
  cloud:
    gateway:
      routes:
      - id: wwl_route
        uri: http://willivie.gitee.io
        predicates:
        - Path=/about
```

#### 方式二：自定义RouteLocator

```java
@Configuration
public class GatewayConfig {
    /**
     * 配置了一个id为route-name的路由规则，
     * 当访问地址 http://localhost:9527/guonei时会自动转发到地址：http://news.baidu.com/guonei
     *
     * @param builder
     * @return
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        RouteLocatorBuilder.Builder routes = builder.routes();
        routes.route("path_route_wwl", r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
        return routes.build();

    }
}
```

### 2、Predicate（断言）

在 Spring Cloud Gateway 中 Spring 利用 Predicate 的特性实现了各种路由匹配规则，有通过 Header、请求参数等不同的条件来进行作为条件匹配到对应的路由。匹配成功则能请求成功，否则报错404.网上有一张图总结了 Spring Cloud 内置的几种 Predicate 的实现。 

![](https://static01.imgkr.com/temp/82d830d5b4724ebd81019f5db489275e.png )

```bash
routes:
 - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
   uri: http://localhost:8001          #匹配后提供服务的路由地址
   predicates:
     - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
     - After=2020-02-05T15:10:03.685+08:00[Asia/Shanghai]    # 断言，路径相匹配的进行路由
    #- Before=2020-02-05T15:10:03.685+08:00[Asia/Shanghai]  # 断言，路径相匹配的进行路由
    #- Between=2020-02-02T17:45:06.206+08:00[Asia/Shanghai],2020-03-25T18:59:06.206+08:00[Asia/Shanghai]
    #- Cookie=username,zzyy
    #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
    #- Host=**.baidu.com
     - Method=GET
     - Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
```

**各种 Predicates 同时存在于同一个路由时，请求必须同时满足所有的条件才被这个路由匹配。** 

> cmd请求测试：curl http://localhost:9527/payment/lb --cookie "name"="wwl"

#### 3、Filter（过滤器）

Spring Cloud Gateway 的 Filter 的生命周期不像 Zuul 的那么丰富，它只有两个：“pre” 和 “post”。

- **PRE**： 这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。
- **POST**：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的 HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。

Spring Cloud Gateway 的 Filter 分为两种：GatewayFilter 与 GlobalFilter。GlobalFilter 会应用到所有的路由上，而 GatewayFilter 将应用到单个路由或者一个分组的路由上。

Spring Cloud Gateway 内置了9种 GlobalFilter，比如 Netty Routing Filter、LoadBalancerClient Filter、Websocket Routing Filter 等，根据名字即可猜测出这些 Filter 的作者，具体参考官网内容：[Global Filters](http://cloud.spring.io/spring-cloud-gateway/single/spring-cloud-gateway.html#_global_filters)

利用 GatewayFilter 可以修改请求的 Http 的请求或者响应，或者根据请求或者响应做一些特殊的限制。 更多时候我们会利用 GatewayFilter 做一些具体的路由配置。

##### yml配置

```bash
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: http://example.org
        filters:
           - AddRequestParameter=X-Request-Id,1024 #过滤器工厂会在匹配的请求头加上一对请求头，名称为X-Request-Id值为1024

```

##### 自定义全局GlobalFilter

这里可以做网关全局日志记录，	统一登录鉴权等。例子：

```java
@Component
public class MyLogGateWayFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        System.out.println("time:" + new Date() + "\t 执行了自定义的全局过滤器: " + "MyLogGateWayFilter" + "hello");

        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname == null) {
            System.out.println("****用户名为null，无法登录");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

### 4、配置动态路由

默认情况下Gateway会根据注册中心注册的服务列表，
以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能

##### application.yml

```bash
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          #    lb://serviceName是spring cloud gateway在微服务中自动为我们创建的负载均衡uri
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
 
 

```

