---
layout:     post
title:      MySQL 关键字 key 、primary key、unique key、index的区别
subtitle:   MySQL keyWord key 、primary key、unique key、index difference
date:       2019-05-13
author:     Monk
header-img: img/Basic_operation_of_MySQL.jpg
catalog: true
tags:
    - MySQL
---

##### MySQL 关键字 key 、primary key、unique key、index的区别
### KEY关键字
key 是数据库的物理结构，它包含两层意义，一是约束（偏重于约束和规范数据库的结构完整性），二是索引（辅助查询用的）。包括primary key, unique key, foreign key 等。
- primary key：有两个作用，一是约束作用（constraint），用来规范一个存储主键和唯一性，但同时也在此key上建立了一个index；
- unique key：也有两个作用，一是约束作用（constraint），规范数据的唯一性，但同时也在这个key上建立了一个index；
- foreign key：也有两个作用，一是约束作用（constraint），规范数据的引用完整性，但同时也在这个key上建立了一个index；

### INDEX 关键字
index是数据库的物理结构，它只是辅助查询的，它创建时会在另外的表空间（mysql中的innodb表空间）以一个类似目录的结构存储。索引要分类的话，分为前缀索引、全文本索引等；因此，索引只是索引，它不会去约束索引的字段的行为。 
例如：create table t(id int, index inx_tx_id (id));

