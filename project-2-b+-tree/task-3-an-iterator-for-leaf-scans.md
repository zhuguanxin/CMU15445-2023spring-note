---
description: 索引迭代器
---

# task#3 An Iterator for leaf scans

## 要求

在完成task#1和task#2中的B+树的实现和全面测试之后，你必须添加一个支持按顺序扫面叶子页面的C++迭代器。基本思想是存储兄弟指针，以便你可以高效地遍历叶子页面。然后实现一个迭代器，按顺序遍历每个叶子页面中的键值对。

你的迭代器必须按照[C++17](https://cplusplus.com/reference/iterator/)风格实现，至少包含以下方法：

1. `isEnd()`返回此迭代器是否指向最后一个键值对。
2. `operator++()`移动到下一个键值对。
3. `operator*()`返回此迭代器当前指向的键值对。
4. `operator==()`返回两个迭代器是否相等。
5. `operator!=()`返回两个迭代器是否不相等。

你的`BPlusTree`还必须实现`begin()`和`end()`方法，以支持在索引上使用C++的for-each循环功能。

## 需要修改的文件

* `src/include/storage/index/index_iterator.h`索引头文件
* `src/index/storage/index_iterator.cpp`相应的源文件
