---
title: Config配置中心
date: 2021-05-22 20:05:55
tags: Config
categories: 
- SpringCloud
---

> 官网：https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/

## 一、概述

##### Config是什么?

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。

##### Config怎么使用？

SpringCloud Config分为**服务端**和**客户端**两部分。

服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

##### Config的特性：

- 集中管理配置文件
- 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
- 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
- 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
- 将配置信息以REST接口的形式暴露

## 二、配置与使用

#### Config服务端配置

##### 1、配置前准备

首先在自己Gitee账号新建仓库springcloud-config，并在仓库下创建config.yml文件

##### 2、pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

##### 3、application.yml

```yaml
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
```

##### 4、主启动

```java
@SpringBootApplication
@EnableConfigServer //激活对配置中心的支持
public class ConfigCenterMain3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```

##### 5、测试通过Config微服务是否可以从Gitee上获取配置内容

输入地址：`http://localhost:3344/master/config/dev`

访问成功后页面上的内容是否与Gitee仓库的文件内容一样就代表配置成功。返回信息如下

```json
{
	"name": "config",
	"profiles": ["dev"],
	"label": null,
	"version": "85a4aa109ccd4c46992a4e867c99b90e3cf56570",
	"state": null,
	"propertySources": [{
		"name": "https://gitee.com/willivie/springcloud-config.git/config-dev.yml",
		"source": {
			"config.info": "master branch,springcloud-config/config-dev.yml version=1"
		}
	}]
}
```

上述的返回的信息包含了配置文件的位置、版本、配置文件的名称以及配置文件中的具体内容，说明server端已经成功获取了git仓库的配置信息。

如果直接查看配置文件中的配置信息可访问：`http://localhost:3344/master/config-dev.yml`，返回：

```yaml
config:
  info: master branch,springcloud-config/config-dev.yml version=1
```

仓库中的配置文件会被转换成web接口，访问可以参照以下的规则：

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

#### Config服务端配置

#### 1、pom.xml

```xml
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

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
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://localhost:3344 #配置中心地址k

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

---

**注： bootstrap.yml是什么？**

`applicaiton.yml` 是用户级的资源配置项
`bootstrap.yml` 是系统级的，优先级更加高

Spring Cloud会创建一个“Bootstrap Context”，作为Spring应用的`Application Context`的父上下文。初始化的时候，`Bootstrap Context`负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的`Environment`。

`Bootstrap`属性有高优先级，默认情况下，它们不会被本地配置覆盖。 `Bootstrap context`和`Application Context`有着不同的约定，所以新增了一个`bootstrap.yml`文件，保证`Bootstrap Context`和`Application Context`配置的分离。

要将Client模块下的application.yml文件改为bootstrap.yml,这是很关键的，
因为bootstrap.yml是比application.yml先加载的。bootstrap.yml优先级高于application.yml

---

##### 3、主启动

```java
@EnableEurekaClient
@SpringBootApplication //无需额外配置
public class ConfigClientMain3355 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3355.class, args);
    }
}
```

##### 4、新增测试类controller

```java
@RestController
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

输入地址访问：`http://localhost:3355/configInfo`，成功显示gitee的配置文件内容

```
master branch,springcloud-config/config-dev.yml version=3
```

成功实现了客户端3355访问SpringCloud Config3344通过Gitee获取配置信息

### 三、Config客户端之动态刷新

**现在存在的问题：**

修改Gitee上的配置文件内容做调整后，刷新config服务端，发现ConfigServer配置中心立刻更新内容，但是ConfigClient客户端没有任何更新，任然是修改前的值。重启config客户端后，就可以获取更新后的内容。

但是每次重启服务来获取更新的配置，难免有点繁琐而且不符合应用程序的设计。那么避免每次更新配置都要重启客户端微服务3355，添加动态刷新配置

##### 1、pom.xml新增依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

##### 2、bootstrap新增依赖

```yam
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

##### 3、controller类新增注解

```java
@RestController
@RefreshScope //新增注解
public class ConfigClientController
{
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}

```

配置结束后，再次修改Gitee上的配置文件内容，需要发送请求命令`curl -X POST "http://localhost:3355/actuator/refresh"`刷新配置。再次获取配置内容发现已经更新成功。这样就达到了，避免服务重启的步骤达到获取更新配置的目的。

##### 想想还有什么问题？

假如有多个微服务客户端3355/3366/3377...等等

每个微服务都要执行一次post请求，手动刷新？

可否广播，一次通知，处处生效？这里引入SpringCloud Bus消息总线，实现一次请求，批量刷新。