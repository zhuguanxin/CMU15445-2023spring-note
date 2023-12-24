---
description: 访问方法执行程序
---

# task#1 Access Method Executors

## 要求

在此任务中，你将实现读取和写入存储系统中表的执行程序。你将在以下文件中完成实现：

* `src/include/execution/seq_scan_executor.h`
* `src/execution/seq_scan_executor.cpp`
* `src/include/execution/insert_executor.h`
* `src/execution/insert_executor.cpp`
* `src/include/execution/update_executor.h`
* `src/execution/update_executor.cpp`
* `src/include/execution/delete_executor.h`
* `src/execution/delete_executor.cpp`
* `src/include/execution/index_scan_executor.h`
* `src/execution/index_scan_executor.cpp`

下面将介绍这些执行程序中的每一个。

## SeqScan 顺序扫描

[`SeqScanPlanNode`](https://github.com/cmu-db/bustub/blob/master/src/include/execution/plans/seq\_scan\_plan.h)可以使用`SELECT * FROM table`语句进行计划。

```sh
bustub> CREATE TABLE t1(v1 INT, v2 VARCHAR(100));
Table created with id = 15
bustub> EXPLAIN (o,s) SELECT * FROM t1;
=== OPTIMIZER ===
SeqScan { table=t1 } | (t1.v1:INTEGER, t1.v2:VARCHAR)
```

SeqScanExecutor迭代遍历表格并逐个返回其元组。

提示：

* 确保你理解在使用`TableIterator`对象时，使用前++和后++运算符之间的区别。如果你混淆了`++iter`和`iter++`，可能会得到奇怪的输出。
* 不要发出在TableHeap中删除的元组。检查每个元组的相应[`TupleMeta`](https://github.com/cmu-db/bustub/blob/master/src/include/storage/table/tuple.h)`is_deleted_` 字段。
* 顺序扫描的输出是每个匹配元组及其原始记录标识符RID的副本。
* BusTub不支持DROP TABLE或者DROP INDEX。你可以通过重新启动shell来重置shell。



### 目标文件

* `bustub/src/include/execution/executors/seq_scan_executor.h`
* `bustub/src/execution/seq_scan_executor.h`

