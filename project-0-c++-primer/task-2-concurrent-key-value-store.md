---
description: 并发键值存储（多线程写时复制字典树）
---

# task#2 Concurrent Key-Value Store

## 要求

将task#1的单线程写时复制字典树改为多线程，实现并发键值存储。在本task中，你需要修改`trie_store.h`和`trie_store.cpp`。该键值存储实现三个功能：

* Get(key):return the value
* Put(key,value):no return value
* Remove(key):no return value

对于task#1的`Trie`类，我们每次修改trie时，都需要获取new root来访问new content。但是对于并发键值存储，`put`和`remove`方法都是没有return value的。这需要你使用并发原语(concurrency primitives)来同步读取和写入，以便在此过程中不会丢失任何数据。

你的并发键值存储应该同时服务于多个reader和一个writer。也就是说，当有人修改trie时，仍然可以对old root进行读操作。当有人正在read时，仍然可以执行写入而无需等待read完成。

此外，如果我们从trie中获得对值的引用，则无论我们如何修改trie，我们都应该能够访问它。Trie中的`Get`函数只返回一个指针。如果存储这个值的trie节点已经被移除，指针会悬空。因此，在TrieStore中，我们返回一个ValueGuard，它既存储了对值的引用，也存储了对应于trie结构根的TrieNode，这样就可以像我们存储ValueGuard一样访问值。

为此，我们在 `trie_store.cpp` 中为您提供了 `TrieStore::Get` 的伪代码。 请仔细阅读并思考如何实现 `TrieStore::Put` 和 `TrieStore::Remove`。

