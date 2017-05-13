---
layout: post
title:  "Fix mysql memory table error:  table xxxx is full"
date:   2017-05-13 23:14:54
categories: mysql
tags: mysql issue
---

* content
{:toc}

记录朋友问及的一个问题，线上的游戏logininfo table采用内存表(memory as storage engin),新用户进不去了，错误信息为：table xxxx is full。

内存表，所使用的内存大小限制由[max_heap_table_size](http://dev.mysql.com/doc/refman/5.1/en/server-system-variables.html#sysvar_max_heap_table_size)确定。

查询数值设定如下，如果没有设定过，这里显示的应该是默认数值：

```shell
mysql> show variables like 'max_heap_table_size';
+---------------------+----------+
| Variable_name       | Value    |
+---------------------+----------+
| max_heap_table_size | 16777216 |
+---------------------+----------+
1 row in set (0.00 sec)
```

修改数值：
```shell
mysql> set  max_heap_table_size=268435456;
Query OK, 0 rows affected (0.00 sec)
mysql> show variables like 'max_heap_table_size';
+---------------------+-----------+
| Variable_name       | Value     |
+---------------------+-----------+
| max_heap_table_size | 268435456 |
+---------------------+-----------+
1 row in set (0.00 sec)
```

`max_heap_table_size`可以动态修改，但对已存在的表无效，通过`alert table`重整表空间。
```shell
mysql> alter table logininfo ENGINE=MEMORY;
Query OK, 1000 rows affected (0.06 sec)
Records: 1000  Duplicates: 0  Warnings: 0
```
