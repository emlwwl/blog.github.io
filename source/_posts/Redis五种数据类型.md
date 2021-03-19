---
title: Redis五种数据类型
date: 2021-02-15 17:05:55
tags: Redis
categories: 
- Redis
---

> Redis的优势之一就是拥有丰富的数据类型，不同的业务场景可以选择最合适的数据类型存储数据。Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。 

> 具体指令可通过 help @string/list/set/hash 查看

> 查看key是哪种类型 type key

### 1、String  (字符串)

- string是redis最基本的类型 ,数据格式为key-value格式。
- string类型可以存储任何数据，应用场景多，需要注意的是，单个key的数据大小不能太大，一般存数据很小的值。
- string类型单个key的值最大能存储512MB数据。 

<!--more-->

```
# 设置键值
127.0.0.1:6379> set name Willivie
OK
# 根据键获取值
127.0.0.1:6379> get name
"Willivie"
# 获取值的长度
127.0.0.1:6379> strlen name
(integer) 8
# 拼接字符串
127.0.0.1:6379> append name ,Grant
(integer) 14
127.0.0.1:6379> get name
"Willivie,Grant"
# 设置时间 set key value EX timeout(秒)
127.0.0.1:6379> set lan Java EX 30
OK
# 查看过期时间：ttl key
127.0.0.1:6379> ttl lan
(integer) 25
127.0.0.1:6379> ttl lan
(integer) 22
# 已设置值，追加过期时间：expire key timeout(秒)
127.0.0.1:6379> expire lan 20
(integer) 1
127.0.0.1:6379> ttl lan
(integer) 16
# 如果不存在则创建
127.0.0.1:6379> setnx sort desc
(integer) 1
127.0.0.1:6379> get sort
"desc"
127.0.0.1:6379> setnx sort asc
(integer) 0
127.0.0.1:6379> get sort
"desc"
# 同时设置多个值
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
# 查看所有key
127.0.0.1:6379> keys *
1) "k2"
3) "k3"
4) "k1"
# 同时获取多个值
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
```

### 2、List （列表）

- List类型是一个链表结构的集合 ，其主要功能有push,pop,range等
- List类型是一个双端链表，可以通过相关的操作集合的头部或者尾部添加和删除元素 
- list主要应用在跟顺序相关的业务中，例如最新消息排行榜，消息队列等。 

```
# 赋值
#  lpush key value ：将一个值插入到列表的头部（最左边）
#  rpush key value ：将一个值插入到列表的尾部（最右边）
127.0.0.1:6379> lpush device socket
(integer) 1
# 取值
# llen key :获取列表的长度
# lindex key index 通过索引获取列表的元素
# lrange key start end 获取指定范围的元素
# start和end代表列表的两个元素下标，获取下标范围内的元素（包括下标）
127.0.0.1:6379> lrange device 0 2
1) "light"
2) "switch"
3) "socket"
```
### 3、Set （集合）

- set中的值是不能重读的！ 

- Redis的set集合是无序不可重复的 
- 集合最大的优势在于可以进行交集并集差集操作 
- set类型根据其集合属性，可以用在，例如关注粉丝业务(判断是否已关注被被关在)，交集查找共同好友，唯一性可以统计访问网站的所有独立IP等。 

```
# 赋值
127.0.0.1:6379> sadd numO 1 2 3
(integer) 3
(1.68s)
# 取值
127.0.0.1:6379> smembers numO
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> sadd numT 3 4 5
(integer) 3
# siff A B 取集合A的差集
127.0.0.1:6379> sdiff numO numT
1) "1"
2) "2"
# 并集
127.0.0.1:6379> sunion key1 key2 
# 返回集合的元素个数
127.0.0.1:6379> scard numO
(integer) 3
# sinter A B 取A和B的交集
127.0.0.1:6379> sinter numO numT
1) "3"
# 判断一个元素是否存在，1存在/0不存在
127.0.0.1:6379> sismember numO 1
(integer) 1
# spop key cuont 随机返回并删除集合内count个元素
127.0.0.1:6379> spop numO 2
1) "3"
2) "2"
# 取set的所有值
127.0.0.1:6379> smembers numO
1) "1"
```

### 4、Hash（哈希）

- hash 是一个 string 类型的key-map! 这个值是一个map集合 
- hash 特别适合用于存储对象，例如用户基本信息等  

```
# 赋值
127.0.0.1:6379> hset user name willivie
(integer) 1
127.0.0.1:6379> hset user age 20
(integer) 1
127.0.0.1:6379> hset user sex boy
(integer) 1
# 取值
127.0.0.1:6379> hget user age
"20"
127.0.0.1:6379> hset user star 1
(integer) 1
# 给map的某个value值自增1
127.0.0.1:6379> hincrby user star 1
(integer) 2
127.0.0.1:6379> hget user star
"2"
# 批量赋值
127.0.0.1:6379> hmset user age 18 name grant
OK
# 批量取值
127.0.0.1:6379> hmget user age name
1) "18"
2) "grant"
# 获取map的所有key
127.0.0.1:6379> hkeys user
1) "name"
2) "age"
3) "sex"
4) "star"
# 获取map所有value
127.0.0.1:6379> hvals user
1) "grant"
2) "18"
3) "boy"
4) "2"
```

### 5、ZSet （有序集合）

- 使用`help @sorted_set`查看相关指令
- zet里元素是有顺序，不能重复的。索引为唯一的，数据却可以重复 
- zset类型主要利用有序和不重复性，例如排序存储班级成绩表，工资表；排行榜应用实现，取Top N 
- 在set的基础上，增加了一个值。`set k1 v1 ； zset k1 score1 v1  `

```
# 赋值
127.0.0.1:6379> zadd type 9 int 3 char 5 bool
(integer) 3
127.0.0.1:6379> zadd type 6 double
(integer) 1
# 取所有值 0 -1代表第一个和最后一个
127.0.0.1:6379> zrange type 0 -1
1) "char"
2) "bool"
3) "double"
4) "int"
# 根据score排序 显示全部的元素 从小到大！
127.0.0.1:6379> zrangebyscore type -inf +inf
1) "char"
2) "bool"
3) "double"
4) "int"
# 从大到进行排序！
127.0.0.1:6379> zrevrange type 0 -1
1) "int"
2) "double"
3) "bool"
4) "char"
# 显示全部的元素和score 从小到大
127.0.0.1:6379> zrangebyscore type -inf +inf withscores
1) "char"
2) "3"
3) "bool"
4) "5"
5) "double"
6) "7"
7) "int"
8) "9"
```



> Redis中，下列三种数据类型的区别

| 数据结构 | 是否可重复 | 是否有序 |   应用场景   |
| :------: | :--------: | :------: | :----------: |
|   List   |     是     |    是    | 时间轴、队列 |
|   Set    |     否     |    否    |  标签、社交  |
|   ZSet   |     否     |    是    |    排行榜    |

> 工具类封装：https://blog.csdn.net/qq_41088297/article/details/109222157