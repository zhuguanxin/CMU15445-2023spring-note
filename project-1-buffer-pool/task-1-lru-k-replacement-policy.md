# task#1 LRU-K Replacement Policy

## 要求

该组件负责跟踪缓冲池中的页面使用情况。你将在`src/buffer/lru_k_replacer.cpp`中实现一个名为`LRUKReplacer`的新类以及其相应的实现文件。请注意`LRUKReplacer`是一个独立的类，与其他Replacer类没有关系。你只需要实现LRU-K替换策略。即使有相应的文件，你也不必实现LRU或者时钟替换策略。

LRU-K算法会淘汰在Replacer中的所有帧中后向k距离最大的帧。后向k距离是当前时间戳和第k次以前访问的时间戳之间的差值。历史访问次数少于k的帧被赋予+inf作为其后向k距离。当多个帧具有+inf的后向k距离时，Replacer将淘汰具有最早整体时间戳（最近记录的访问是所有帧中最近的访问）的帧。

