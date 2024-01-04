---
description: 锁管理器
---

# task#1 Lock Manager

为了确保事务操作的正确交错执行，数据库管理系统DBMS使用锁管理器(Lock Manager,LM)来控制事务对数据项的访问。锁管理器的基本思想是维护一个关于活动事务当前持有的锁的内部数据结构。事务在访问数据项之前向锁管理器发出锁请求，锁管理器将根据情况授予锁、阻塞事务直到锁可用或者中止事务。

在你的实现中，BusTub系统将使用一个全局LM。TableHeap类和Executor类将使用你的锁管理器，在事务尝试访问或修改元组时，根据记录ID（Record ID，RID）获取对元组记录的锁。

你的锁管理器必须实现层次化的表级和元祖级锁，以及三个隔离级别：READ\_UNCOMMITED（读未提交）、READ\_COMMITED（读已提交）和REPEATABLE\_READ（可重复读）。锁管理器应根据事务的隔离级别来授予或释放锁。

我们提供了一个事务上下文句柄（`include/concurrency/transaction.h`），带有隔离级别属性（即`READ_UNCOMMITED` 、 `READ_COMMITTED` 和 `REPEATABLE_READ` ），以及关于以获取锁的信息。锁管理器需要检查事务的隔离级别，并在锁定/解锁请求上提供正确的行为。任何无效的锁操作都应导致事务状态为ABORTED（隐式中止）并抛出异常。对锁的尝试失败（死锁）不会引发异常，但是锁管理器应返回锁请求的false。

## 要求

你需要修改的唯一文件是`LockManager` 类，相关的文件为：

* `concurrency/lock_manager.cpp`
* `include/concurrency/lock_manager.h`

你必须实现以下函数：

* `LockTable(Transaction, LockMode, TableOID)`
* `UnlockTable(Transction, TableOID)`
* `LockRow(Transaction, LockMode, TableOID, RID)`
* `UnlockRow(Transaction, TableOID, RID, force)`

LM的具体锁定机制取决于事务的隔离级别、表级或元祖级锁以及所设计的锁类型。确保你熟悉`Transaction` 类的API和成员变量，这些变量在`transaction.h`和`lock_manager.h`中定义。然后请仔细阅读`[LOCK_NOTE]` 、 `[UNLOCK_NOTE]` 和锁管理器函数的规范 （in lock\_manager.h ）

有一个force参数，UnlockRow因为执行器实现可能需要确定元组是否可访问，然后才能决定是否包含它。如果force设置为true，则该操作将绕过所有2PL检查，就像该元组没有被锁定一样。

对于 `UnlockRow` ，我们有一个 `force` 参数，因为在执行器实现中，我们可能需要查看事务是否可以访问元组，然后再决定是否将此元组扫描到父执行器。

提示：

* 我们建议您在尝试添加与死锁处理相关的任何功能之前，成功实施并彻底测试没有死锁检测的锁管理器。
* 您将需要某种方法来跟踪哪些操作正在等待锁定。请看一下 lock\_manager.h 中的 `LockRequestQueue` 类。
* 仔细考虑何时需要升级锁，以及需要更新表/元组锁时需要对锁 `LockRequestQueue` 执行哪些操作。
* 当锁可用时，您将需要某种方式来通知等待的交易。我们建议将 `std::condition_variable` provided 用作 `LockRequestQueue` 的一部分。
* 锁管理器应根据需要更新事务的状态。例如，事务的状态可能由 `unlock` 操作从 `GROWING` 更改为 `SHRINKING` 。请参阅中 transaction.h 的方法
* 您应该跟踪事务使用的 `*_lock_set_` 锁，以便 `TransactionManager` 可以在提交或中止事务时适当地释放锁。
* 将事务的状态设置为“已中止”会隐式中止它，但在调用之前 `TransactionManager::Abort` 不会显式中止。您应该阅读此函数和提供的测试，以了解它的作用以及锁管理器在中止过程中的使用方式。

## 加锁提示

* 通用行为：LockTable()和LockRow()都是阻塞方法；它们应该等待锁被授予，然后返回。如果此期间事务被中止，则不授予锁并返回false。
* 多个事务：LM应该为每个资源维护一个队列；应该以先进先出的方式向事务授予锁。如果有多个兼容的锁请求，则只要遵守先进先出原则，就应该同时授予所有请求的锁。
* 支持的锁模式：
  * 表级锁应支持所有锁模式
  * 行级锁不应支持意向锁。如果尝试这样做，则应将TransactionState设置为ABORTED并抛出TransactionAbortExeception(ATTEMPTED\_INTENTION\_LOCK\_ON\_ROW)
* 隔离级别：根据隔离级别，事务应尝试获得锁，但是仅在必要时且仅在允许的情况下。
* 例如
