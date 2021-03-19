---
title: Centos-7 安装mysql
date: 2021-02-22 20:05:55
tags: Linux
categories: 
- Linux
---

#### 第一步、前往mysql官网下载所需的版本

> Mysql5.7的rpm包的[下载地址](https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.22-1.el7.x86_64.rpm-bundle.tar ) ，这里的版本是5.7.22

下载完成后就上传的CentOS系统上。

#### 第二步、解压安装

<!--more-->

```bash
//创建目录
cd /opt
mkdir software
cd software
mkdir mysql
上传文件
[root@youxi2 ~]# tar -vxf mysql-5.7.27-1.el7.x86_64.rpm-bundle.tar　　//将解压的文件放到Mysql目录下
[root@youxi2 ~]# yum -y install make gcc-c++ cmake bison-devel ncurses-devel libaio libaio-devel net-tools　　//安装依赖包
由于CentOS7开始自带的数据库是mariadb，所以需要卸载系统中的mariadb组件，才能安装mysql的组件
[root@youxi2 ~]# rpm -qa | grep mariadb
mariadb-libs-5.5.60-1.el7_5.x86_64
[root@youxi2 ~]# yum -y remove mariadb-libs
　　现在开始安装mysql，由于依赖关系，所以顺序是固定的。
　　
//一个一个按顺序安装
[root@youxi2 ~]# rpm -ivh Mysql/mysql-community-common-5.7.16-1.el7.x86_64.rpm
警告：Mysql/mysql-community-common-5.7.16-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-common-5.7.16-1.e################################# [100%]
   
[root@youxi2 ~]# rpm -ivh mysql-community-libs-5.7.27-1.el7.x86_64.rpm
警告：Mysql/mysql-community-libs-5.7.16-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-libs-5.7.16-1.el7################################# [100%]
   
   
[root@youxi2 ~]# rpm -ivh mysql-community-libs-compat-5.7.27-1.el7.x86_64.rpm
警告：Mysql/mysql-community-libs-compat-5.7.16-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中... ################################# [100%]
正在升级/安装...
1:mysql-community-libs-compat-5.7.1################################# [100%]

[root@youxi2 ~]# rpm -ivh /opt/software/mysql/mysql-community-client-5.7.27-1.el7.x86_64.rpm 
警告：Mysql/mysql-community-client-5.7.16-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-client-5.7.16-1.e################################# [100%]
   
[root@youxi2 ~]# rpm -ivh mysql-community-server-5.7.27-1.el7.x86_64.rpm　　//之后安装就成功了
警告：Mysql/mysql-community-server-5.7.16-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-server-5.7.16-1.e################################# [100%]
```

这里记录一下，我这里在第五步安装community-server时报错

```bash
[root@localhost software]# rpm -ivh mysql-community-server-5.7.22-1.el7.x86_64.rpm 
警告：mysql-community-server-5.7.22-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
错误：依赖检测失败：
        /usr/bin/perl 被 mysql-community-server-5.7.22-1.el7.x86_64 需要
        perl(Getopt::Long) 被 mysql-community-server-5.7.22-1.el7.x86_64 需要
        perl(strict) 被 mysql-community-server-5.7.22-1.el7.x86_64 需要

```

问题原因很简单：缺少perl.x86_64这个依赖，使用yum安装即可。 

` yum -y install perl.x86_64 `

总之，仔细看报错信息！ 

#### 第三步、启动mysql并设置开机自启

```bash
[root@youxi2 ~]# systemctl start mysqld
[root@youxi2 ~]# systemctl enable mysqld
[root@youxi2 ~]# systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2019-06-02 12:11:34 CST; 45s ago
 Main PID: 7840 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─7840 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

6月 02 12:11:26 youxi2 systemd[1]: Starting MySQL Server...
6月 02 12:11:34 youxi2 systemd[1]: Started MySQL Server.
```

#### 第四步、获取mysql临时密码，设置mysql的root用户密码

###### 注意：5.6的密码使用 cat /root/.mysql_secret 查询

5.7版本看以下内容

```bash
[root@youxi2 ~]# grep "password" /var/log/mysqld.log　　//前往日志文件查找临时密码
2019-06-02T04:11:28.935057Z 1 [Note] A temporary password is generated for root@localhost: zS+u&ro49wbo
别忘了。。记一下   _kizdUj_E77a

[root@youxi2 ~]# mysql -uroot -p"zS+u&ro49wbo"
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.16

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

//可以看到设置简单的密码是会报错的，因为密码有安全级别，具体原因看最后的扩展
mysql> alter user 'root'@'localhost' identified by '123456';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> alter user 'root'@'localhost' identified by 'root1234';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> alter user 'root'@'localhost' identified by 'root1234ABC';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> alter user 'root'@'localhost' identified by 'root1234ABCD!@#$';
Query OK, 0 rows affected (0.00 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
mysql> exit
Bye
　　修改密码出来使用“alter user 'root'@'localhost' identified by 'root1234ABCD!@#$';”，也可以使用“set password for root@localhost=password('root1234ABCD!@#$');”
```

#### 第五步、测试

由于有特殊符号，必须用引号包裹密码

```bash
[root@youxi2 ~]# mysql -u root -p'root1234ABCD!@#$'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.16 MySQL Community Server (GPL)
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
```

#### 扩展：如果想要设置简单密码概如何操作？

有两种方法，一种在mysql里使用命令修改，一种直接修改配置文件。

##### 在mysql里使用命令修改的办法：

```bash
[root@youxi2 ~]# mysql -u root -p'root1234ABCD!@#$'

mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.16 MySQL Community Server (GPL)
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select @@validate_password_policy;　　//这个参数是密码复杂程度
+----------------------------+
| @@validate_password_policy |
+----------------------------+
| MEDIUM                     |
+----------------------------+
1 row in set (0.02 sec)

mysql> select @@validate_password_length;　　//这个参数是密码长度
+----------------------------+
| @@validate_password_length |
+----------------------------+
|                          8 |
+----------------------------+
1 row in set (0.00 sec)
mysql密码简单就这两句命令

mysql> set global validate_password_policy=0;　　//global全局的
Query OK, 0 rows affected (0.02 sec)

mysql> set global validate_password_length=1;
Query OK, 0 rows affected (0.00 sec)

mysql> set password for root@localhost=password('root');

mysql> flush privileges;　　//刷新
Query OK, 0 rows affected (0.00 sec)

mysql> exit　　//退出
Bye

[root@youxi2 ~]# mysql -uroot -p1234
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.16 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

##### 说明：

1.`validate_password_policy`复杂度级别：0表示密码达到长度即可；1表示密码需达到长度，还需有数字、大小写字母（可以单一可以混合）以及特殊字符；2表示密码需达到长度，还需数字、大小写字母（可以单一可以混合）以及特殊字符字典文件。MEDIUM是中等，也就是1。

2.`validate_password_length`其实是一个动态的值，它的最小值等于`validate_password_number_count`+`validate_password_special_char_count(2*validate_password_mixed_case_count)`，而这三个参数分别对应密码中数字、特殊字符、大小写字母的最小数量。我操作时设置了`validate_password_length=1`，实际再次读取`validate_password_length`的值是4。

```
mysql> select @@validate_password_length;
+----------------------------+
| @@validate_password_length |
+----------------------------+
|                          4 |
+----------------------------+
1 row in set (0.00 sec)
```

##### 直接修改配置文件的办法：

```bash
[root@youxi2 ~]# vim /etc/my.cnf
validate-password=OFF　　//在[mysqld]模块内添加，将validate_password插件关闭
[root@youxi2 ~]# systemctl restart mysqld　　//重启mysqld服务
[root@youxi2 ~]# mysql -uroot -p1234    
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.16 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set password for root@localhost=password('1');　　//validate_password插件关闭后密码长度只需大于等于1即可，复杂度没有要求
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;　　//刷新
Query OK, 0 rows affected (0.00 sec)

mysql> exit　　//退出
Bye
[root@youxi2 ~]# mysql -uroot -p1
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.16 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

注意：Mysql5.7是自带validate_password插件，关闭后对密码没有复杂度要求，只需密码长度大于等于1。

建议：/etc/my.cnf中将默认字符集设置为utf8，即添加一行character_set_server=utf8，然后重启mysqld

#### 修改远程访问权限

```bash
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select host,user from user;
+-----------+---------------+
| host      | user          |
+-----------+---------------+
| localhost | mysql.session |
| localhost | mysql.sys     |
| localhost | root          |
+-----------+---------------+
3 rows in set (0.00 sec)

mysql> update user set host='%' where user='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> exit

//重启mysql服务
systemctl restart mysqld
```