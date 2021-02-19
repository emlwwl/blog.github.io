---
title: Java8中的forEach()方法
date: 2020-09-13 20:05:55
tags: Java
categories: 
- Java
--- 

下面介绍了两种集合的常用遍历方式。废话不多说，直接上代码

<!--more-->

* HashMap的使用

```java
Map<String,Integer> map = new HashMap<>();
map.put("AA",1);
map.put("BB",2);
map.put("CC",3);
//普通遍历map的方法之一
for(String key : map.keySet()){
    System.out.println(key);
}
for(String value : map.values()){
    System.out.println(value);
}
    
//用forEach()
map.forEach((key,value)->System.out.println(key+" or "+value);

//加上判断后
items.forEach((k,v)->{
    System.out.println("Item : " + k + " Count : " + v);
    if("E".equals(k)){
        System.out.println("Hello E");
    }
});

```


* List的使用

```
List<String> list = new ArrayList<>();
list.add("张三");
list.add("李斯");
list.add("王武");

//普通遍历的方式
for(String name:list){
    System.out.println("姓名："+name);
}

//用forEach的方式
list.forEach(name->System.out.println("姓名："+name));
//参考
list.forEach(System.out::println);
//filter和forEach
list.stream()
    .filter(name->name.contains("张三")
    .forEach(System.out::println);
```

