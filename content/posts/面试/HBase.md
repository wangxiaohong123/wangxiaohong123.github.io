---
title: HBase
tags:
  - 面试专题
categories: 面试专题
copyright: true
---



他是一个基于HDFS的nosql。强一致性，适用于海量数据的简单增删改查。

hbase里每行数据都有一个rowkey，表里的数据按照rowkey排序。每列都有自己的列族，猎德命名就是列族:列名，比如order:base，每个数据的值都有自己的时间戳，一个时间戳下的一个单元格数据就是一个cell。他是列式存储，大概是这样：

rowkey			timestamp				列				值

order_1_001	t1							order:base	xxxx

order_1_001	t2							order:detail	yyyy

