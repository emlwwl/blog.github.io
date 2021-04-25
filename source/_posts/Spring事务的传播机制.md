---
title: Spring事务的传播机制
date: 2021-03-15 10:05:00
tags: Spring
categories: 
- Java
---

### 传播机制：

| 类型                                                  | 说明                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| @Transactional(propagation=Propagation.REQUIRED)      | 如果有事务，那么加入事务,，没有的话新建一个(默认情况下)      |
| @Transactional(propagation=Propagation.NOT_SUPPORTED) | 容器不为这个方法开启事务                                     |
| @Transactional(propagation=Propagation.REQUIRES_NEW)  | 不管是否存在事务，都创建一个新的事务。原来的挂起，新的执行完毕，继续执行老的事务 |
| @Transactional(propagation=Propagation.MANDATORY)     | 必须在一个已有的事务中执行，否则抛出异常                     |
| @Transactional(propagation=Propagation.NEVER)         | 必须在一个没有的事务中执行，否则抛出异常(与Propagation.MANDATORY相反) |
| @Transactional(propagation=Propagation.SUPPORTS)      | 如果其他bean调用这个方法，在其他bean中声明事务，那就用事务。如果其他bean没有声明事务，那就不用事务 |



<!--more-->