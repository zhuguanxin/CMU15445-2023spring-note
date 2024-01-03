---
description: 排序+限制执行程序和topN优化
---

# Sort+Limit Executors and Top-N Optimization

## 要求

最后，你需要实现一些更常见的执行程序，并在以下文件中完成实现：

* `src/include/execution/sort_executor.h`
* `src/execution/sort_executor.cpp`
* `src/include/execution/limit_executor.h`
* `src/execution/limit_executor.cpp`
* `src/include/execution/topn_executor.h`
* `src/execution/topn_executor.cpp`
* `src/optimizer/sort_limit_as_topn.cpp`

在开始此任务之前，`IndexScanExecutor`必须在task#1中实现。如果表上有索引，查询处理层将自动选取该索引进行排序。在其他情况下，你将需要一个特殊的排序执行器来执行此操作。

对于所有order by子句，我们假设每个排序键都只出现依次。你无需担心排序中的ties。

## Sort 排序

如果查询的order by属性与索引的键不匹配，BusTub将生成一个SortPlanNode查询，例如：

```sql
EXPLAIN SELECT * FROM __mock_table_1 ORDER BY colA ASC, colB DESC;
```

此计划节点的输出与输入模式相同。你可以从中提取排序键`order_bys`，然后`std::sort`使用自定义比较器对子节点的元组进行排序。你可以假设表中的所有条目都完全适合内存。

如果查询不包含排序方向（`ASC`/`DESC`），则排序模式将为default（`ASC`）。

## Limit 限制

指定LimitPlanNode查询将生成的元组数。请看以下示例：

```sql
EXPLAIN SELECT * FROM __mock_table_1 LIMIT 10;
```

限制LimitExecutor其子执行程序的输出元组数。如果其子执行程序生成的元组数小于计划节点中指定的限制，则此执行程序不起作用，并生成它接收的所有元组。

此计划节点的输出方案与其输入模式相同。你不需要支持偏移量。

## TopN Optimization Rule Top-N优化规则

最后，你应该修改BusTub的优化器，以有效地支持前N个查询。

```sql
EXPLAIN SELECT * FROM __mock_table_1 ORDER BY colA LIMIT 10;
```

默认情况下，BusTub会将查询以以下方式进行查询：

* 对表中的所有数据进行排序
* 获取前10个元素，这显然是低效的，因为查询只需要最小的值。更智能的做法是动态跟踪到目前为止最小的10个元素。
* 需要将order by+limit子句转换为TopN executor

### 优先队列 priority\_queue

* [set/map/priority\_queue实现自定义排序](https://blog.csdn.net/SYaoJun/article/details/106976964)
* 使用堆动态维护前N个数据，超过N个时，弹出不满足要求的

### 目标文件

* `src/executor/topn_executor.cpp`
* `src/include/executor/topn_executor.h`
* `src/optimizer/sort_limit_as_topn.cpp`

### 思路

* 成员函数：
  1. Constructor()
  2. Init()：在初始化阶段就需要把前N个元素准备好
  3. Next()
* 优化器：
  * 将sort和limit算子转换为topn算子

