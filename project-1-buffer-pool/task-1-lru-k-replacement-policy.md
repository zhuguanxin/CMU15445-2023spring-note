# task#1 LRU-K Replacement Policy

## 要求

该组件负责跟踪缓冲池中的页面使用情况。你将在`src/buffer/lru_k_replacer.cpp`中实现一个名为`LRUKReplacer`的新类以及其相应的实现文件。请注意`LRUKReplacer`是一个独立的类，与其他Replacer类没有关系。你只需要实现LRU-K替换策略。即使有相应的文件，你也不必实现LRU或者时钟替换策略。

LRU-K算法会淘汰在Replacer中的所有帧中后向k距离最大的帧。后向k距离是当前时间戳和第k次以前访问的时间戳之间的差值。历史访问次数少于k的帧被赋予+inf作为其后向k距离。当多个帧具有+inf的后向k距离时，Replacer将淘汰具有最早整体时间戳（最近记录的访问是所有帧中最近的访问）的帧。

* 优先淘汰距离最大的帧
* 少于k次访问 && 距离是+inf，优先被淘汰
* 当有多个+inf时，优先淘汰整体时间戳最早的

你需要实现在本课程中讨论的LRU-K策略。你需要按照头文件（`src/include/buffer/lru_k_replacer.h`）和源文件（`src/buffer/lru_k_replacer.cpp`）中定义的方式实现以下方法：

* `Evict(frame_id_t* frame_id)`：淘汰具有与Replacer追踪的所有其他可淘汰帧相比具有最大后向k距离的frame。将frame id存储在输出参数中并返回True。如果没有可淘汰的frame，则返回false。
* `RecordAccess(frame_id_t frame_id)` ：记录给定的frame id在当前时间戳被访问。此方法应在页面在`BufferPoolManager`中固定之后调用。
* `Remove(frame_id_t frame_id)`：移除与frame相关联的所有访问历史记录。此方法仅在`BufferPoolManager`中删除页面时调用。
* `SetEvictable(frame_id_t frame_id, bool set_evictable)`：此方法控制frame是否可淘汰。它还控制`LRUKReplacer`的大小。当你实现`BufferPoolManager`时，将会知道何时调换此函数。具体而言，当页面的pin count达到0时，其对应的frame将会被标记为evictable，并增加Replacer的大小。
* `Size()`：此方法返回当前在`LRUKReplacer`可淘汰的帧数。

## LRU-K原理

