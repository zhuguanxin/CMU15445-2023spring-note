---
description: 并发查询执行
---

# task#3 Concurrent Query Execution

## 要求

在并发查询执行期间，执行器需要适当地对元组进行加锁/解锁，以实现对应事务中指定的隔离级别。为了简化这个任务，你可以忽略并发索引执行，只专注于表元组。

你需要更新在P3中实现的一些执行器（顺序扫描/插入/删除）的Next()方法。请注意，当锁定/解锁失败时，事务应中止。如果事务中止，您将需要撤消其先前的写入操作。为此，您需要维护每个事务中的写入集，该写入集由事务管理器 `Abort()` 的方法使用。如果执行器未能获取锁，则应抛出一个`ExecutionException` 锁，以便执行引擎通知用户查询失败。

不应假定事务仅包含一个查询。具体来说，这意味着一个元组可能被事务中的不同查询多次访问。考虑在不同的隔离级别下应如何处理此问题。

若要完成此任务，必须在以下执行程序和事务管理器中添加对并发查询执行的支持：

* `src/execution/seq_scan_executor.cpp`
* `src/execution/insert_executor.cpp`
* `src/execution/delete_executor.cpp`
* `src/concurrency/transaction_manager.cpp`

## 思路

* 读操作：seq-scan顺序扫描
  1. 根据不同隔离级别提供不同并发性能
  2. 表IS/行S
  3. 在seq-scan执行器中不同隔离级别的锁处理方式
     * 读未提交：不加锁（加锁有开销），每次读最新的数据记录（存在脏读问题）
     * 读提交：加S锁（一个事务中两次读取结果不一致，不可重复读），每次读完就立即释放（strict-2PL）
     * 可重复读(rigorous-2PL)
* 写操作：insert\&delete
  1. 修改必须互斥
  2.  表IS/行X

