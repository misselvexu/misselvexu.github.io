---
layout:     post
title:      "[Database] Lock & Isolation & Transaction For MySQL"
subtitle:   "9 types of locks, 3 types of reads, 4 isolation levels"
date:       2022-09-27 02:59:59
author:     "Palmer Xu"
header-img: "img/in-post/ssr-bg.jpg"
catalog: true
tags:
    - Database
    - Isolation
    - Transaction
    - MySQL
---

> Code is Not Only Code .


## 场景
商城订单系统中，一个账户的充值行为与消费扣款行为属于两种比较常见的操作，在时间和空间上都存在着并发执行的可能。
所以如果对一个账户同时去执行充值和支付扣款的行为，会发生什么？以及这些问题数据库本身是如何解决的？那我们应用程序又应做些什么？

### 概述部分
1. 什么是排他锁？什么是共享锁？
2. 什么是行锁、表锁、意向锁、间隙锁、临键(next-key)锁？
3. 什么是悲观锁、乐观锁？
4. 什么是『两阶段锁』协议？
5. 什么是LCBB（锁并发控制协议）？
6. 什么是MVCC（多版本控制协议）？
7. 什么是脏读、不可重复读、幻读？
8. RU、RC、RR、SE隔离级别？

- 问题一：你知道这一堆锁之间有啥关系吗？
- 问题二：MVCC如何解决事务读的问题的？

## 锁之间的关系
假设前提：不考虑数据库引擎、隔离级别，简单表格做数据记录和读取。

### 场景一

<img width="848" alt="image" src="https://user-images.githubusercontent.com/3436472/192413232-7ea1ac1d-9c79-4725-a7c2-d1cd7df013bd.png">

解析：
过程称为脏读，（场景一中T1、T2都是先查后更新操作，更新时无保护行为）

引导问题：
在不修改数据库隔离级别的情况下，是否可以通过SQL手动解决脏读问题？

<img width="1033" alt="image" src="https://user-images.githubusercontent.com/3436472/192413360-fa243c9a-dd42-4555-88cf-ce550f6ac823.png">

解析：
1. 过程中《手动添加排他锁》的操作，称为行锁。（行锁的本质是在索引节点上加锁）
2. 表锁还可以通过执行lock table的语句来获取。

引导问题：
事务A申请行锁，事务B申请表锁，B会被阻塞吗？

3. 与『排他锁』对应的就是『共享锁』，也称为『读写锁 』。

引导问题：
- 排他锁与共享锁的场景选择？
- 排他锁（X）与共享锁（S）什么时候释放？

## 『两阶段锁』协议
- 加锁阶段：在该阶段可以进行加锁操作。在对任何数据进行读操作之前要申请并获得 S 锁（共享锁，其它事务可以继续加共享锁，但不能加排它锁），在进行写操作之前要申请并获得 X 锁（排它锁，其它事务不能再获得任何锁）。加锁不成功，则事务进入等待状态，直到加锁成功才继续执行。
- 解锁阶段：当事务释放了一个封锁以后，事务进入解锁阶段，在该阶段只能进行解锁操作不能再进行加锁操作。

引导问题：
两阶段锁协议可以避免死锁吗？

### 场景二
FOR UPDATE 属于一种『悲观锁』，那对应的用乐观锁咋解决上述的脏读的问题？
// JD：乐观锁场景比较简单，就不细化了

小结一下
前面对于脏读问题的解决，我们通过与数据库隔离级别无关的方法来进行解决的，实际的业务场景比我们的演示的场景要复杂的很多，那数据库是如何解决这些问题的呢？

回顾：前文中脏读的发生对应的隔离级别是，读未提交（RU）。

引导问题：
1、读未提交的隔离级别下，有锁吗？
2、那『读已提交』，能解决脏读问题吗？是如何解决的？

目前主要有两种方式去实现RC级别下的脏读问题。
- LBCC 基于锁的并发控制（Lock-Based Concurrency Control)
- MVCC 基于多版本的并发控制协议 (Multi-Version Concurrency Control)

总结一下：
- 查：只查当前事务之前的记录，或者回滚版本比当前大的已删记录
- 增：加新版本的记录
- 删：把老记录标记上回滚版本
- 改：本质上是加新记录， 同时把老记录标上回滚版本

## （补充）ReadView运作机制：

概念：
- m_ids：生成ReadView时，有哪些还未提交的事务，记录下这些事务的id;  
- min_trx_id：m_ids中最小的事务id；  
- max_trx_id：InnoDB下一个要生成的事务id;  
- creator_trx_id：生成当前ReadView的事务id;

工作原理（多事务并发情况下的数据隔离）：
- 若被访问版本的trx_id = creator_trx_id，表示当前事务在访问自己修改过的记录，所以该版本可以被当前事务访问。
- 若被访问版本的trx_id < min_trx_id，表示生成该版本的事务在当前事务生成ReadView前已经提交，所以该版本可以被当前事务访问。
- 若被访问版本的trx_id >= 的max_trx_id，表明生成该版本的事务在当前事务生成ReadView后才开启，所以该版本不可以被当前事务访问。
- 若被访问版本的 trx_id 在min_trx_id和max_trx_id之间，那就需要判断一下trx_id是不是在m_ids列表中，如果在，说明创建ReadView时生成该版本的事务还是活跃的，该版本不可以被访问；如果不在，说明创建ReadView时生成该版本的事务已经被提交，该版本可以被访问。


引导问题：
1、MVCC 机制下， 什么是快照读，什么是当前读？


## 事务的并发问题与隔离级别
### 脏写
事务A更新数据后，未提交，B事务修改同一条数据，A若回滚，导致B修改的值没了，出现了脏写。
<img width="773" alt="image" src="https://user-images.githubusercontent.com/3436472/192413399-f02d4811-f019-47f6-8024-19b4a3f8460b.png">


### 脏读
事务A更新数据后，未提交，B事务读取数据为a，事务A若回滚，导致B再读取已经读取不到a了，出现了脏读。

<img width="813" alt="image" src="https://user-images.githubusercontent.com/3436472/192413418-41c38a3e-13c8-40ee-a97e-c22e259529a3.png">


无论是脏写还是脏读，都是因为一个事务去更新或者查询了另外一个还没提交的事务更新过的数据。因为另外一个事务还没提交，所以它随时可能会回滚，必然导致你更新或查询到的数据就没了，这就是脏写和脏读两种场景。

### 不可重复读
事务A读取了数据，之后事务B修改了数据并提交，事务A再次读取时，值发生了变化，两次读取的数据不一致。
<img width="777" alt="image" src="https://user-images.githubusercontent.com/3436472/192413427-4e0a2089-b34b-4970-b35b-78cc4c53bdc3.png">


### 幻读
事务A做范围查询，事务B插入或删除了几条数据并提交，则A再次查询时数据发生了变化。
<img width="848" alt="image" src="https://user-images.githubusercontent.com/3436472/192413434-e53a208d-e5e6-423e-9abf-5e18b3a3659c.png">


总结上边介绍的事务问题，包括两种：
- 多个事务同时对缓存页里的一行数据进行更新，导致的问题（脏写）
- 多个事务并发执行的时候，有的事务在对一行数据做更新，有的事务在查询这行数据，导致问题（脏读、不可重复读、幻读）


## 扩展ReadView + MVCC解决事务并发的场景

- 读已提交情况
<img width="1065" alt="image" src="https://user-images.githubusercontent.com/3436472/192413449-58bbef2e-248c-4597-87b0-c1fdf541ff66.png">


- 可重复读情况
<img width="897" alt="image" src="https://user-images.githubusercontent.com/3436472/192413458-ac058c0d-b617-4fec-96ce-7bd87bb5a3d1.png">


