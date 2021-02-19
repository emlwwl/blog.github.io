---
title: SpringBoot数据传输加密
date: 2020-10-29 10:05:00
tags: Java
categories: 
- Java
---

### 引入jar包
```
 <dependency>
	 <groupId>com.cxytiandi</groupId>
	 <artifactId>monkey-api-encrypt-core</artifactId>
	 <version>1.2.RELEASE</version>
  </dependency>
```
### 启动类加注解 EnableEncrypt
```java
@EnableEncrypt
@SpringBootApplication(exclude = {SecurityAutoConfiguration.class})
@EnableTransactionManagement
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
<!--more-->

### 增加加密的key配置：

```bash
spring:
	encrypt:
		key: *************
		debug: false
```
- spring.encrypt.key：加密key，必须是16位
- spring.encrypt.debug：是否开启调试模式,默认为false,如果为true则不启用加解密操作
  **注意**：dev配置文件为ture，prod配置文件为false。这样本地就不会开启，调式的时候不会启用加解密，线上启用就可以了。

------------
 **备注1** 

- 到这一步已经开启了加解密全局配置，默认所有接口都会对接口请求和响应数据进行加解密
- 如果在某个方法上使用Decrypt和Encrypt注解，则是只有使用了注解的方法会加解密


### 基于注解来做控制：@Decrypt @Encrypt
使用这两个注解后，全局加解密就失效了
@Encrypt 对响应数据进行加密操作
```java
 @ApiOperation(value = "测试", response = PopTmpDTO.class, notes = "")
    @GetMapping("/all")
    @Encrypt
    public ApiResponses<List<PopTmpDTO>> getTest() {
        List<PopTmpDTO> list = popTmpService.list(new LambdaQueryWrapper<>()).stream().map(e->e.convert(PopTmpDTO.class)).collect(Collectors.toList());
        return success(list);
    }
```

@Decrypt 对请求数据进行解密操作
```java
@ApiOperation(value = "测试2", notes = "willivie 2019-12-29：新增接口<br />")
    @PostMapping("/create")
    @Decrypt
    @Encrypt
    public ApiResponses<PopTmpPARM> create(@RequestBody
           @Validated(PopTmpPARM.Create.class) PopTmpPARM popTmpPARM) {
        popTmpPARM.setPopName("改变测试！！");
        return success(popTmpPARM);
    }
```
 **备注2** 
- 注意：加了Encrypt和Decrypt注解后，类似GetMapping注解一点要在后面加路径，比如@GetMapping → @GetMapping（"/list")，不然会报错!如果默认全局加解密的情况则不会出现这种情况

###默认开启全部加解密功能，如果想要忽略某些接口怎么办？
配置方式可以使用下面的方式进行忽略：
方法①：在application配置中加：（注意yml配置方式）
```java
spring.encrypt.requestDecyptUriIgnoreList[0]=/save
spring.encrypt.responseEncryptUriIgnoreList[0]=/encryptEntity
spring.encrypt.responseEncryptUriIgnoreList[1]=/save
```
方法②：在具体方法上加：@DecryptIgnore和@EncryptIgnore，即可生效
```java
@ApiOperation(value = "测试2", notes = "willivie 2019-12-29：新增接口<br />")
    @PostMapping("/create")
    @DecryptIgnore
    @EncryptIgnore
    public ApiResponses<PopTmpPARM> create(@RequestBody
           @Validated(PopTmpPARM.Create.class) PopTmpPARM popTmpPARM) {
        popTmpPARM.setPopName("改变测试！！");
        return success(popTmpPARM);
    }
```

**备注3** 
- 加密忽略：responseEncryptUriIgnoreList 或者 @EncryptIgnore
- 解密忽略：requestDecyptUriIgnoreList 或者 @DecryptIgnore

>参考项目：<https://github.com/yinjihuan/monkey-api-encrypt>