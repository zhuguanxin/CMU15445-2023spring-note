# task#1 LRU-K Replacement Policy

## 要求

该组件负责跟踪缓冲池中的页面使用情况。你将在`src/buffer/lru_k_replacer.cpp`中实现一个名为`LRUKReplacer`的新类以及其相应的实现文件。请注意`LRUKReplacer`是一个独立的类，与其他Replacer类没有关系。你只需要实现LRU-K替换策略。即使有相应的文件，你也不必实现LRU或者时钟替换策略。

LRU-K算法会淘汰在Replacer中的所有帧中后向k距离最大的帧。后向k距离是当前时间戳和第k次以前访问的时间戳之间的差值。历史访问次数少于k的帧被赋予+inf作为其后向k距离。当多个帧具有+inf的后向k距离时，Replacer将淘汰具有最早整体时间戳（最近记录的访问是所有帧中最近的访问）的帧。

你需要实现在本课程中讨论的LRU-K策略。你需要按照头文件（`src/include/buffer/lru_k_replacer.h`）和源文件（`src/buffer/lru_k_replacer.cpp`）中定义的方式实现以下方法：

* `Evict(frame_id_t* frame_id)`：淘汰具有与Replacer追踪的所有其他可淘汰帧相比具有最大后向k距离的frame。将frame id存储在输出参数中并返回True。如果没有可淘汰的frame，则返回false。
* `RecordAccess(frame_id_t frame_id)` ：记录给定的frame id在当前时间戳被访问。此方法应在页面在`BufferPoolManager`中固定之后调用。
* `Remove(frame_id_t frame_id)`：移除与frame相关联的所有访问历史记录。此方法仅在`BufferPoolManager`中删除页面时调用。
* `SetEvictable(frame_id_t frame_id, bool set_evictable)`：此方法控制frame是否可淘汰。它还控制`LRUKReplacer`的大小。当你实现`BufferPoolManager`时，将会知道何时调换此函数。具体而言，当页面的pin count达到0时，其对应的frame将会被标记为evictable，并增加Replacer的大小。
* `Size()`：此方法返回当前在`LRUKReplacer`可淘汰的帧数。

## LRU-K原理

参考阅读：

* [LRU-K和2Q缓存算法介绍](https://www.jianshu.com/p/c4e4d55706ff)
* Leetcode：[146.LRU缓存](https://leetcode.cn/problems/lru-cache/)
* SIGMOD1993：[The LRU-K Page Replacement Algorithm For Database Disk Buffering](https://www.cs.cmu.edu/\~natassa/courses/15-721/papers/p297-o\_neil.pdf)

LRU(Least Recently Used)，K表示最近使用次数。[LRU](https://zhuanlan.zhihu.com/p/161269766)是一种内存数据淘汰策略，常用于当内存不足时，淘汰最近最少使用的数据。LRU-K则是需要维护两个队列，用于记录所用缓存数据被访问的历史。

* 优先淘汰距离最大的帧
* 少于k次访问 && 距离是+inf，优先被淘汰
* 当有多个+inf时，优先淘汰整体时间戳最早的

## 阅读原文件

首先看`lru_k_replacer_test.cpp`:

```cpp
TEST(LRUKReplacerTest, DISABLED_SampleTest) {
  LRUKReplacer lru_replacer(7, 2);

  // Scenario: add six elements to the replacer. We have [1,2,3,4,5]. Frame 6 is non-evictable.
  lru_replacer.RecordAccess(1);
  lru_replacer.RecordAccess(2);
  lru_replacer.RecordAccess(3);
  lru_replacer.RecordAccess(4);
  lru_replacer.RecordAccess(5);
  lru_replacer.RecordAccess(6);
  lru_replacer.SetEvictable(1, true);
  lru_replacer.SetEvictable(2, true);
  lru_replacer.SetEvictable(3, true);
  lru_replacer.SetEvictable(4, true);
  lru_replacer.SetEvictable(5, true);
  lru_replacer.SetEvictable(6, false);
  ASSERT_EQ(5, lru_replacer.Size());

  // Scenario: Insert access history for frame 1. Now frame 1 has two access histories.
  // All other frames have max backward k-dist. The order of eviction is [2,3,4,5,1].
  lru_replacer.RecordAccess(1);

  // Scenario: Evict three pages from the replacer. Elements with max k-distance should be popped
  // first based on LRU.
  int value;
  lru_replacer.Evict(&value);
  ASSERT_EQ(2, value);
  lru_replacer.Evict(&value);
  ASSERT_EQ(3, value);
  lru_replacer.Evict(&value);
  ASSERT_EQ(4, value);
  ASSERT_EQ(2, lru_replacer.Size());

  // Scenario: Now replacer has frames [5,1].
  // Insert new frames 3, 4, and update access history for 5. We should end with [3,1,5,4]
  lru_replacer.RecordAccess(3);
  lru_replacer.RecordAccess(4);
  lru_replacer.RecordAccess(5);
  lru_replacer.RecordAccess(4);
  lru_replacer.SetEvictable(3, true);
  lru_replacer.SetEvictable(4, true);
  ASSERT_EQ(4, lru_replacer.Size());

  // Scenario: continue looking for victims. We expect 3 to be evicted next.
  lru_replacer.Evict(&value);
  ASSERT_EQ(3, value);
  ASSERT_EQ(3, lru_replacer.Size());

  // Set 6 to be evictable. 6 Should be evicted next since it has max backward k-dist.
  lru_replacer.SetEvictable(6, true);
  ASSERT_EQ(4, lru_replacer.Size());
  lru_replacer.Evict(&value);
  ASSERT_EQ(6, value);
  ASSERT_EQ(3, lru_replacer.Size());

  // Now we have [1,5,4]. Continue looking for victims.
  lru_replacer.SetEvictable(1, false);
  ASSERT_EQ(2, lru_replacer.Size());
  ASSERT_EQ(true, lru_replacer.Evict(&value));
  ASSERT_EQ(5, value);
  ASSERT_EQ(1, lru_replacer.Size());

  // Update access history for 1. Now we have [4,1]. Next victim is 4.
  lru_replacer.RecordAccess(1);
  lru_replacer.RecordAccess(1);
  lru_replacer.SetEvictable(1, true);
  ASSERT_EQ(2, lru_replacer.Size());
  ASSERT_EQ(true, lru_replacer.Evict(&value));
  ASSERT_EQ(value, 4);

  ASSERT_EQ(1, lru_replacer.Size());
  lru_replacer.Evict(&value);
  ASSERT_EQ(value, 1);
  ASSERT_EQ(0, lru_replacer.Size());

  // This operation should not modify size
  ASSERT_EQ(false, lru_replacer.Evict(&value));
  ASSERT_EQ(0, lru_replacer.Size());
}
```

图示：



<figure><img src="../.gitbook/assets/lru_k_replacer_test.svg" alt=""><figcaption></figcaption></figure>
