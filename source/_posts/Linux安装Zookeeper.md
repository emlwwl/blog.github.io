---
title: Zookeeper 注册中心
date: 2021-05-07 20:05:55
tags: Zookeeper
categories: 
- SpringCloud
---
### 一、安装前环境配置

ZooKeeper是用Java编写的，运行在Java环境上，因此，在部署zk的机器上需要安装Java运行环境。为了正常运行zk，我们需要JRE1.6或者以上的版本。 

### 二、下载并解压Zookeeper

zookeeper下载链接：https://archive.apache.org/dist/zookeeper

解压：`tar -zvxf zookeeper-3.4.13.tar.gz`

### 三、zookeeper配置

进入conf目录

<!--more-->

```
[root@localhost local]# cd zookeeper-3.4.13/conf
```

zoo.cfg文件配置 **不配做会报错**

```
[root@localhost conf]# cp  zoo_sample.cfg  zoo.cfg
```

其他配置

```
[root@localhost conf]# mkdir /tmp/zookeeper
[root@localhost conf]# mkdir /tmp/zookeeper/data
[root@localhost conf]# mkdir /tmp/zookeeper/log
```

```
dataDir=/tmp/zookeeper/data
dataLogDir=/tmp/zookeeper/log
server.1=192.168.120.200:2888:3888
#如果要配置集群
server.2=192.168.120.200:2888:3888
server.3=192.168.120.200:2888:3888
....
```

环境变量

```
[root@localhost zookeeper-3.4.13]# export ZOOKEEPER_INSTALL=/usr/local/zookeeper-3.4.13/
[root@localhost zookeeper-3.4.13]# export PATH=$PATH:$ZOOKEEPER_INSTALL/bin
```

### 四、启动

启动zookeeper服务端，bin目录下

```
[root@localhost bin]# ./zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.11/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

启动zookeeper客户端，bin目录下

```
[root@localhost bin]# ./zkCli.sh
Connecting to localhost:2181
.....
WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] 
```

### 五、项目集成zookeeper客户端

1、pom.xml    **注意jar包冲突**

```xml
<!-- SpringBoot整合Web组件 -->
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper3.5.3-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.4.11版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.11</version>
        </dependency>
```

2、application.yml

```
#8004表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 8004
#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 192.168.120.200:2181
```

3、Main.java

```
@SpringBootApplication
@EnableDiscoveryClient //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
public class PaymentMain8004 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8004.class,args);
    }
}
```

4、启动8004项目后，进入zkServer.sh查询注册的实例

```
[zk: localhost:2181(CONNECTED) 4] ls /services               [cloud-provider-payment]
[zk: localhost:2181(CONNECTED) 5] ls /services/cloud-provider-payment
[fedd0ce2-45b6-49a3-a796-53b5b7571dc0]
```

看到上面的实例名称，即代表注册到zookeeper服务端成功