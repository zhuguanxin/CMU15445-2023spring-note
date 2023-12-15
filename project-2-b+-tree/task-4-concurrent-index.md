---
description: 并发索引
---

# task#4 Concurrent index

## 要求

最后，修改B+树实现，使其安全地支持并发操作。你应该使用课堂和教科书中描述的latch crabbing技术。遍历索引的线程应根据需要获取B+树页面上的锁，以确保安全的并发操作，并应在确定这样做是安全的时候尽快释放父页上的锁。

建议你首先将FetchPageBasic更改为FetchPageWrite和FetchPageRead来完成此任务，具体取决于你是要访问具有读取权限还是写入权限的页面。然后修改你的实现，以根据需要获取和释放读写锁存器，以实现latch crabbing算法。

注意：切勿在单个线程中两次获取相同的读锁定。这可能会导致死锁。

##
