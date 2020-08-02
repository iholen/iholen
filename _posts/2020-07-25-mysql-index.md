---
layout: post
title:  "Mysql索引"
date:   2020-07-25 14:12:05 +0800
categories: database
author: heoller
introduction: Mysql索引
---

### 索引数据结构
索引就是排好序的数据结构。

* 索引数据结构不使用二叉树的原因:
    * 树的高度高，I/O次数过多
    * 二叉树可能会退化成链表
* 索引数据结构不使用红黑树的原因:
    * 数据量大的时候，树的高度不可控，I/O次数还是多
* Hash索引优缺点
    * 查询速度更快
    * 只支持等值和IN查询，不支持范围查询
    * 存在Hash冲突问题
* B-Tree数据结构:<br>
    ![B-Tree](/iholen/assets/images/B-Tree.png)
* B+Tree数据结构:<br>
    ![B+Tree](/iholen/assets/images/B+Tree.png)
* B-Tree和B+Tree的区别:
    * B+Tree非叶子节点不存储数据(优点: 每页(16K)可以存储更多的索引)，存在冗余索引。
    * B+Tree叶子节点直接通过指针连接，更好的支持区间访问。
    * B+Tree叶子节点存储所有索引字段。
* MyISAM存储引擎使用的是非聚集索引，InnoDB使用的是聚集索引(叶子节点包含完整数据)。
* MyISAM索引叶子节点存储的是数据在磁盘的地址。
* InnoDB表为什么推荐整型自增的主键？
    * 整型比较的性能更好
    * 自增可以减少页的合并和分裂
* 为什么辅助索引的叶子节点存储主键？
    * 可以节省空间，因为主键索引的叶子节点已经存储了所有数据
    * 保证数据的一致性，不需要同时维护多个索引的叶子节点数据
* 联合索引的最左前缀原则

### 索引小记
##### 联合索引(a,b,c)

* a=1 and b>4 and c=5
    * 只用到(a,b), 由于b是范围查询，c用不到

* a=1 and b like 'k%kk%' and c=5
    * 用到(a,b,c), 因为`k%`相当于等值匹配，mysql底层使用了**索引下推**
```mysql
# 由key_len可知a,b,c都用到了
mysql> explain select * from s1 where col_d='a' and col_e like "k%kk%" and col_f='f';
+----+-------------+-------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | s1    | NULL       | range | idx_key_part  | idx_key_part | 909     | NULL |    1 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```
* a=1 and b like '%kk%' and c=5
    * 只用到(a), 因为`%kk%`相当于范围查询，b和都c用不到
```mysql
# 由key_len可知只有a用到了
mysql> explain select * from s1 where col_d='a' and col_e like "%kk%" and col_f='f';
+----+-------------+-------+------------+------+---------------+--------------+---------+-------+------+----------+-----------------------+
| id | select_type | table | partitions | type | possible_keys | key          | key_len | ref   | rows | filtered | Extra                 |
+----+-------------+-------+------------+------+---------------+--------------+---------+-------+------+----------+-----------------------+
|  1 | SIMPLE      | s1    | NULL       | ref  | idx_key_part  | idx_key_part | 303     | const |    1 |   100.00 | Using index condition |
+----+-------------+-------+------------+------+---------------+--------------+---------+-------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```

##### `limit`最佳实践
```
select * from users join (select id from users limit 10000,20) u on u.id = users.id;
```
##### `join`
inner join: 驱动表为数据量小的表， left join: 驱动表为左边的表， right join: 驱动表为右边的表

* 嵌套循环连接Nested-Loop Join(NLJ)算法<br>
    join字段为索引字段时，会使用该算法，扫描次数 N + N (表大小-t1: N, t2: M, M > N)
* 基于块的嵌套循环连接Block Nested-Loop Join(BNL)算法<br>
    join字段为非索引字段时，会使用该算法，扫描次数为N + M (表大小-t1: N, t2: M, M > N), 且在内存(join_buffer，默认大小256k)中的判断次数为N * M
##### `order by`
如果`order by`没有走索引，`extra`列的值会有use filesort选项，不会体现在key_len值上，某些情况可以通过使用覆盖索引解决没有命中的问题。
##### `group by`
group和order类似，会先排序再分组，可以使用order by null禁用group的排序。
##### filesort(文件排序)
Mysql通过`max_length_for_sort_data`系统变量(默认值1024B)决定排序方式，字段总长度小于这个系统变量就使用单路排序，否则，使用双路排序。

* 单路排序
将查询结果加载到内存中进行排序，会占用较大内存。
* 双路排序
将结果中的id和需要排序的字段加载到内存中排序，排序后再根据id回表查询最终结果。

### 索引设计原则
1. 代码先行，索引后上
2. 联合索引尽量覆盖查询条件，范围查询的字段往后建
3. 大字符串字段可以考虑前缀索引
4. 优先满足where条件 