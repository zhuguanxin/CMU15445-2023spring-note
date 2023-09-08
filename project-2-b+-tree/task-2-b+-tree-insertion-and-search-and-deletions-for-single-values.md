# task#2 B+ Tree Insertion & Search & Deletions for Single Values

## task#2a要求

对于checkpoint#1，你的B+树索引必须支持单个值的插入(Insert())和搜索(GetValue())。该索引应只支持唯一键；如果尝试重新插入已存在的键到索引中，它不应执行插入操作，并返回false（重复key返回false）。

如果插入操作违反了B+树的不变性，应该对B+树的页进行分割（或重新分配键）。如果插入操作导致根页的page id发生变化（插入分裂，树的层数增加），你必须更新B+树索引的头页中的`root_page_id`。你可以通过访问构造函数中提供的`header_page_id_`来完成这个操作。然后，通过使用reinterpret\_cast，你可以将该页强转为`BPlusTreeHeaderPage`(from `src/include/storage/page/b_plus_tree_header_page.h`)，并且从哪里更新根页的page id。你还必须实现`GetRootPageId`方法，该方法当前默认返回0。

我们建议你使用project#1的page guard类来帮助处理同步问题。对于这个checkpoint，我们建议在访问页面时使用`FetchPageBasic`(defined in `src/include/storage/page/`)。当你稍后在task#4中实现并发控制时，可以根据需要将其更改为使用`FetchPageRead`和`FetchPageWrite`。

你可以选择使用Context类(defined in `src/include/storage/index/b_plus_tree.h`)来跟踪你已读取或写入的page（via the `read_set_` and `write_set_` fields），或者存储其他需要递归传递到其他函数中的元数据。

如果你正在使用`Context` class，以下是一些建议：

* 你可能只需要在插入和删除时使用 `write_set_` 。根据你的实现方式，可能不需要使用`read_set_`。
* 你可能希望在Context中存储root page的page id，并在修改B+树时获取header page的write guard。
* 要找到当前节点的父节点，可以查看`write_set_`的末尾，它应该包含沿访问路径的所有节点。
* 你应该使用`BUSTUB_ASSERT`来帮助你找到实现中的不一致数据。例如，如果你要拆分一个节点（除了根节点），你可能希望确保`write_set_`中仍然至少有一个节点。如果你需要拆分根节点，你可能还希望检查`header_page_`是否为`std::nullopt`。
* 要解锁header page，只需将`header_page_`设置为`std::nullopt`。要解锁其他页面，从`write_set_`中pop并drop。

B+树是基于任意key、value和key comparator类型进行参数化的。我们定义了一个宏`INDEX_TEMPLATE_ARGUMENTS`，用于为你生成正确的模板参数声明。

```cpp
template<typename KeyType,
        typename ValueType,
        typename KeyComparator
>
```

这些参数类型包括：

* `KeyType`: index中每个key的类型。实际上，这将是一个`GenericKey`。`GenericKey`的实际大小是可变的，并且由其自己的模板参数指定，该参数取决于索引属性的类型。
* `ValueType`: index中每个value的类型。实际上，这将是一个64位的RID。
* `KeyComparator`: 一个用于比较两个`KeyType`实例之间大小的比较器类。这些类将包含在`KeyType`的实现文件中。

注意：我们的B+树函数还接受一个默认值为nullptr的Transaction\*参数。这是为了project#4准备的，如果你想在并发控制中实现并发索引查找，可以使用该函数。在本项目中，通常不需要使用该参数。

