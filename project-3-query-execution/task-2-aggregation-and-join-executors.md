---
description: 聚合和联接执行程序
---

# task#2 Aggregation & Join Executors

## 要求

在此任务中，你将添加一个聚合执行器和多个连接执行器，并使优化器能够在规划查询时在嵌套循环联接和哈希联接之间进行选择。你将在以下文件中完成实现：

* `src/include/execution/aggregation_executor.h`
* `src/execution/aggregation_executor.cpp`
* `src/include/execution/nested_loop_join_executor.h`
* `src/execution/nested_loop_join_executor.cpp`
* `src/include/execution/hash_join_executor.h`
* `src/execution/hash_join_executor.cpp`
* `src/optimizer/nlj_as_hash_join.cpp`

下面介绍每项任务。

## Aggregation 聚集

`AggregationPlanNode`支持如下的查询：

```sql
EXPLAIN SELECT colA, MIN(colB) FROM __mock_table_1 GROUP BY colA;
EXPLAIN SELECT COUNT(colA), min(colB) FROM __mock_table_1;
EXPLAIN SELECT colA, MIN(colB) FROM __mock_table_1 GROUP BY colA HAVING MAX(colB) > 10;
EXPLAIN SELECT DISTINCT colA, colB FROM __mock_table_1;
```

请注意，聚合执行器本身不需要处理`having`谓词。计划其将`having`是为`FilterPlanNode`。聚合执行器只需要为输入的每个组执行聚合。它只有一个子节点。

聚合的模式是先分组列，然后是聚合列。

如[lecture#10](https://15445.courses.cs.cmu.edu/fall2022/slides/)所述，实现聚合的常见策略是使用哈希表。在本项目中，你将使用这种方法，但是我们做出了简化假设，即聚合哈希表完全适合内存。这意味着你
