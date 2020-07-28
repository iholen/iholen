---
layout: post
title:  "Mysql优化之Explain"
date:   2020-07-28 20:54:05 +0800
categories: database
author: heoller
introduction: Mysql优化之Explain
---

* 执行计划中开启展示延伸表
```mysql
mysql> set session optimizer_switch='derived_merge=off';
Query OK, 0 rows affected (0.00 sec)
```
### 通过列子理解type列(性能从上到下依次递减)
* `system`(表中确定只有一条记录匹配时) & `const`(primary key 或者 unique key 与常数比较时)
```mysql
mysql> explain select * from (select * from s1 where id=1) temp;
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table      | partitions | type   | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
|  1 | PRIMARY     | <derived2> | NULL       | system | NULL          | NULL    | NULL    | NULL  |    1 |   100.00 | NULL  |
|  2 | DERIVED     | s1         | NULL       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)
```
* eq_ref(primary key 或者 unique key 做关联时)
```mysql
mysql> explain select * from films left join users on films.user_id = users.id;
+----+-------------+-------+------------+--------+---------------+---------+---------+------------------------+------+----------+-------------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                    | rows | filtered | Extra       |
+----+-------------+-------+------------+--------+---------------+---------+---------+------------------------+------+----------+-------------+
|  1 | SIMPLE      | films | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                   |    1 |   100.00 | NULL        |
|  1 | SIMPLE      | users | NULL       | eq_ref | PRIMARY       | PRIMARY | 8       | learning.films.user_id |    1 |   100.00 | Using where |
+----+-------------+-------+------------+--------+---------------+---------+---------+------------------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```
* ref(普通索引或者唯一索引的部分前缀做等值匹配时)
```mysql
mysql> explain select * from users where users.name = 'abc';
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | users | NULL       | ref  | idx_name      | idx_name | 32      | const |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
* range
```mysql
mysql> explain select * from films where id > 1;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | films | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL |    1 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.01 sec)
```
* index(扫描二级索引的所有叶子节点，即满足覆盖索引条件)
```mysql
mysql> explain select * from users;
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | users | NULL       | index | NULL          | idx_name | 32      | NULL |    1 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
* ALL(扫描聚簇索引的所有叶子节点)
```mysql
mysql> explain select * from films;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | films | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```
### Extra列
* `Using index`: 使用了覆盖索引
* `Using temporary`: 使用了临时表处理查询
* `Using filesort`: 文件排序，小于设置的`sort buffer`时在内存排序，否则在磁盘完成排序。
### Tips
联合索引(a,b,c)

* a=1 and b>4 and c=5
    * 只用到(a,b), 由于b是范围查询，c用不到

* a=1 and b like 'k%kk%' and c=5
    * 用到(a,b,c), 因为`k%`相当于等值匹配
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
