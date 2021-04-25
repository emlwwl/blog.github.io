---
title: MySQL常用命令
date: 2020-09-13 20:05:55
tags: MySQL
categories: 
- MySQL
---

### 一、常用语句

1、查看mysql版本号

 `select version();` 

2、查看当前登录的用户

 `select user();` 

3、查看当前数据库

 `select database();` 

<!--more-->

4、查看当前时间，返回年月日时分秒

 `select now();` 

5、查看日期的时间戳

 `select unix_timestamp('2019-7-20');`

6、显示数据表的结构

`describe 表名; `

7、查看表的创建细节

`show create table 表名; `

8、运行如下语句查看卡死的线程，有个时间字段可以看出卡住了多长时间  

`select * from information_schema.innodb_trx;`

9、运行如下语句可杀死线程，全部杀死后，数据库恢复正常   

`kill trx_mysql_thread_id`

10、查询连接池连接数  

`show full processlist`

11、查看最大连接数

`show variables like '%max_connections%';`

12、修改最大连接数(mysql重启后会失效)  

`set GLOBAL max_connections = 8800;`

### 二、常用操作，增删改查

1、查询表

```
查询所有列
select * from stu;
查询指定列
select sid,sname,age from stu;
表别名
select * from product as p;
列别名
select pname as pn from product;
用来去除重复数据，是对整个结果集进行数据重复抑制的，而不是针对某一列。
select distinct Department,SubCompany from Employee;
字段间计算
select age*salary,name from employee;
运算查询
select pname,price+10 from product;
comm列有很多记录的值为NULL，因为任何东西与NULL相加结果还是NULL，所以结算结果可能会出现NULL。下面使用了把NULL转换成数值0的函数IFNULL
 select *,sal+ifnull(comm,0) from emp;
```

2、修改表

```
alter table 表名 add 列名 类型（长度） [约束]; --添加列
alter table 表名 modify 列名 类型（长度） [约束]; --修改列的类型长度及约束
alter table 表名 change 旧列名 新列名 类型（长度） [约束]; --修改表列名
alter table 表名 drop 列名; --删除列
alter table 表名 character set 字符集; --修改表的字符集
rename table 表名 to 新表名; --修改表名
```

3、新增数据 (列名与列值的类型，个数，顺序要一一对应。 )

```
insert into 表(列名1，列名2，列名3..) values(值1，值2，值3..);--向表中插入某些列
insert into 表 values(值1，值2，值3..); --向表中插入所有列
```

4、修改数据

```
update 表名 set 字段名=值,字段名=值...; --这个会修改所有的数据，把一列的值都变了
update 表名 set 字段名=值,字段名=值... where 条件;
```

5、删除数据

```
delete from 表名  --删除表中所有记录
delete from 表名 where 条件
truncate table 表名;
```

