---
description: 并发键值存储（多线程写时复制字典树）
---

# task#2 Concurrent Key-Value Store

## 要求

将task#1的单线程写时复制字典树改为多线程，实现并发键值存储。在本task中，你需要修改`trie_store.h`和`trie_store.cpp`。该键值存储实现三个功能：

* Get(key):return the value
* Put(key,value):no return value
* Remove(key):no return value

