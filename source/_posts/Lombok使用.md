---
title: Lombok的使用
date: 2021-04-08 17:05:55
tags: Lombok
categories: 
- Java
---

#### 1、@AllArgsConstructor

##### 说明

使用后添加一个构造函数，该构造函数含有所有已声明字段属性参数。

生成全参构造函数。

<!--more-->

##### 示例

```
//这里注释这个注解就会报错
@AllArgsConstructor
public class User {
    private String name;
    private int age;

    public static void main(String[] args) {
        User user = new User("name", 18);
    }
}
```

#### 2、@Builder

##### 说明

生成静态Builder方法

在设计模式中的思想是：用一个内部类去实例化 一个对象，避免一个类出现过多构造函数

##### 示例

```
@Builder
public class User {
    private String name;
    private int age;

    public static void main(String[] args) {
        User user = User.builder()
                .name("name")
                .age(18).build();
    }
}
```

#### 3、@Data

##### 说明

集成：@Getter/@Setter，@ToString，@EqualsAndHashCode，@RequiredArgsConstructor。 

##### 示例

```
@Data
public class User {
    private String name;
    private int age;

    public static void main(String[] args) {
        User user = new User();
        user.setName("name");
        user.setAge(18);
        System.out.println(user.toString());
    }
}
```

#### 4、@Accessors

##### @Accessors(chain=true)

###### 说明
链式访问，该注解设置chain=true，生成setter方法返回this（**也就是返回的是对象**），代替了默认的返回void。 

###### 示例

```
@Data
@Accessors(chain=true)
public class User {
    private String name;
    private Integer age;

    public static void main(String[] args) {
        //开起chain=true后可以使用链式的set
        User user=new User().setAge(31).setName("pollyduan");//返回对象
        System.out.println(user);
    }

}
```



##### @Accessors(fluent=true)

###### 说明

在chain=true的基础上，getter和setter不带set和get前缀。

###### 示例

```
@Data
@Accessors(fluent=true)
public class User {
    private String name;
    private Integer age;

    public static void main(String[] args) {
        //fluent=true开启后默认chain=true，故这里也可以使用链式set
        User user=new User().age(18).name("name");//不需要写set
        System.out.println(user);
    }

}
```

##### @Accessors(prefix="f")

###### 说明

使用set方法忽略指定的前缀。不推荐这样去命名。 

###### 示例

```
@Data
public class User {
    @Accessors(prefix = "f")
    private String fName;
    private int age;

    public static void main(String[] args) {
        User user = new User();
        //注意方法名
        user.setName("name");
        user.setAge(18);
        System.out.println(user.toString());
    }
}
```

