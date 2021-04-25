---
title: MySQL事务
date: 2021-04-13 20:05:55
tags: MySQL
categories: 
- MySQL
---

## 一、简介

1、在MySQL中只有Innodb存储引擎的数据库才支持事务

2、事务处理可以用来维护数据的完整性，保证一组数据操作，要么全部成功，要么全部失败

3、事务主要用来管理insert，update，delete语句

<!--more-->

## 二、事务的基本要素（ACID）

1、原子性（Atomicity）：事务开始后的所有操作，**要么全部完成，要么全部失败**,一旦在某个环节发生错误，之前执行的操作会被**回滚** （Rollback）到事务开始前的状态。

2、一致性（Consistency）：在事务开始之前和结束之后，数据库的完整性没有被破坏。**比如A向B转账，不可能A扣了钱，B却没收到。** 

3、隔离性（Isolation）：同一时间，只允许一个事务请求同一条数据，不同的事务之前彼此没有任何干扰。

4、持久性（Durability）:事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚

## 三、事务的实现原理

mysql没执行一条聚聚记录一条日志

1、start transaction，先记个日志，真正执行。

## 四、事务的相关指令

**事务的开始**

`begin`或`start trancaction`都是显式开启一个事务；

**事务的提交**

`commit`或`commit work` 都是等价的

**事务回滚**

`rollback`或`rollback word`也是等价的

**示例**:

```
start transaction; #开启事务 还可以用begin开启事务
UPDATE user set balance = balance - 200 where id = 1; 
UPDATE user set balance = balance + 200 where id = 2;
commit; 提交事务，代表事务结束。更新的数据保存到数据库

start transaction; #开启事务
UPDATE user set balance = balance - 200 where id = 1; 
UPDATE user set balance = balance + 200 where id = 2; rollback; 事务回滚，上面更新的数据将被还原到执行前
```

## 五、事务的并发问题

1、脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据

2、不可重复读：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果不一致。

3、幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。

**小结：不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表**

## 六、事务的隔离级别

**MySQL的默认隔离级别是可重复读**

| 事务隔离级别                 | 脏读 | 不可重复读 | 幻读 |
| ---------------------------- | ---- | ---------- | ---- |
| 读未提交（Read Uncommitted） | 是   | 是         | 是   |
| 读已提交（Read Committed）   | 否   | 是         | 是   |
| 可重复读（Repeatable Read）  | 否   | 否         | 是   |
| 串行化（Serializable）       | 否   | 否         | 否   |