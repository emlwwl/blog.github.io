---
title: Arrays.asList()踩坑
date: 2021-03-24 13:05:55
tags: Java
categories: 
- Java
---
### 一、问题

**把数组转换成集合时,不能使用其修改集合相关的方法？**

**阿里巴巴java开发规范**说到使用工具类Arrays.asList()方法把数组转换成集合时,不能使用其修改集合相关的方法,它的add/remove/clear方法会抛出UnsupportedOperationException() ，直接看代码

<!--more-->

```java
    String strs = "[1,2,3]";
    String strip = StringUtils.strip(strs,"[]");
    System.out.println("strip = " + strip);
    String[] split = strip.split(",");
    List<String> list = Arrays.asList(split);
    list.add("4");
```

在编译的情况下，List集合的add方法弹出警告，运行也真的报错了，情况如下

![](https://static01.imgkr.com/temp/b8ab421e5c3243f7bb5d33938fdce7de.png )

### 二、分析

点击看报错的类，发现这个ArrayList并不是我们平时用的ArrayList.。而是Arrays里面的一个内部类.而且这个内部类没有add,clear,remove方法,所以抛出的异常其实来自于AbstractList. 

```
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    /**
     * Sole constructor.  (For invocation by subclass constructors, typically
     * implicit.)
     */
    protected AbstractList() {
    }
 }
```

### 三、解决

其实很简单，用new ArrayList嵌套到外面就可以了，这时就是使用的我们平时的ArrayList，用add和remove方法也都不会报错了。

```java
//修改前
List<String> list = Arrays.asList(split);
list.add("4");
//修改后
List<String> list = new ArrayList(Arrays.asList(split));
list.add("4");
```

