---
layout: post
title:  "Mysql索引数据结构"
date:   2020-07-25 14:12:05 +0800
categories: database
author: heoller
introduction: Mysql索引数据结构
---

### 索引
索引就是排好序的数据结构。

* 索引数据结构不使用二叉树的原因:
    * 树的高度高，I/O次数过多
    * 二叉树可能会退化成链表
* 索引数据结构不使用红黑树的原因:
    * 数据量大的时候，树的高度不可控，I/O次数还是多
* B-Tree数据结构:<br>
    ![B-Tree](/iholen/assets/images/B-Tree.png)
* B+Tree数据结构:<br>
    ![B+Tree](/iholen/assets/images/B+Tree.png)