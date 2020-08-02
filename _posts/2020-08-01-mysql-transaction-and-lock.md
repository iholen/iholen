---
layout: post
title:  "Mysql事务隔离级别和锁"
date:   2020-08-01 23:24:05 +0800
categories: database
author: heoller
introduction: Mysql事务隔离级别和锁
---

## 事务
ACID属性(原子性，一致性，隔离性，持久性)

* 并发事务可能带来的问题
    * 更新丢失：事务之间相互覆盖结果
    * 脏读：读到其他事务未提交的数据
    * 不可重复读：一次事务中，读到其他事务已提交的变更结果
    * 幻读：一次事务中，读到其他事务已提交的新增数据
* 事务隔离级别
    * 读未提交(READ-UNCOMMITTED)
    * 读已提交(READ-COMMITTED)
    * 可重复读(REPEATABLE-READ)
        没有解决幻读问题，在对新数据执行一次更新操作时，会感知到新增的数据。 
    * 可串行化(SERIALIZABLE)

查看/设置事务隔离级别：
```mysql
mysql> show global variables like 'tx_isolation';
mysql> set tx_isolation='repeatable-read';
```
## 锁
* 乐观锁和悲观锁
    * 乐观锁：通过版本号进行判断数据是否有被修改过，没有锁等待，类似CAS。
    * 悲观锁：事务之间会相互等待
* 读锁(共享锁)和写锁(排他锁)
    * 读锁
    * 写锁 
* 表锁和行锁
    * 表锁：一般会在数据迁移的时候使用，保证数据不会变。
    * 行锁：锁住单行记录，可能会产生死锁(大部分情况mysql会自动检测到死锁，并回滚产生死锁的事务)。InnoDB的行锁是针对索引的，如果更新语句的条件是非索引字段，行锁会升级为表锁。
* 间隙锁
> 锁住记录之间以及值所在的区间所有的行，在可重复读隔离级别下生效。尽量避免间隙锁。
以下示例中，mysql会锁住(5,10]以及(10, 20]区间的所有行。其中(3,20]这个区间也叫**临键锁**。
```mysql
mysql> select * from users;
+----+---------+
| id | name    |
+----+---------+
|  4 | heoller |
|  5 | wjy     |
| 10 | wjy1    |
| 20 | wjy2    |
+----+---------+
4 rows in set (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update users set name='aaa' where id > 8 and id < 15;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

MyISAM存储引擎不支持事务；执行查询时，会在表上加读锁；非读操作时，会在表上加写锁，且不支持行锁