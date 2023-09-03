# task#2 Buffer Pool Manager

## 要求

接下来，实现缓冲池管理器。缓冲池管理器负责从`DiskManager`中获取数据库页面并将它们存储在内存中。`BufferPoolManager`还可以在当被明确要求这样做或者为新页腾出空间时，淘汰脏页。

为了确保你的实现与系统的其余部分一起正常工作，我们将为你提供一些已经完成的功能。你也不需要实现实际读取和写入磁盘数据的代码(`DiskManager`)。

系统中所有内存页面都由`Page`表示。`BufferPoolManager`不需要知道`Pages`的内容。作为系统开发人员，你需要知道`Page`只是缓冲池中内存的容器，因此并不特定于唯一的`page`。换句话说，每个Page`对象`都包含一个内存块，`DiskManager`将使用该内存块作为位置来复制它从磁盘读取的物理页面的内容。`BufferPoolManager`会重用同一个`Page`对象来存储数据，因为它在磁盘上移动。这意味着在系统的生命周期中，同一个`Page`会包含不同的物理页面。`Page`的标识符`page_id`跟踪它包含的物理页面。若`Page`对象不包含物理页面，`page_id`将被设置为`INVALID_PAGE_ID`。

每个`Page`对象还未`pinned`该页面的线程数维护一个计数器。你的`BufferPoolManger`不允许释放`pinned`页面。每个`Page`还会记录它是否脏页。你的工作是记录一个`pinned page`是否被修改。你的`BufferPoolManager`必须先把脏页的内容写回磁盘，然后才能重用该对象。

你的`BufferPoolManager`将会用到此前实现的`LRUKReplacer`类。`LRUKReplacer`会追踪`Page`对象何时被访问，以便它可以决定淘汰哪个对象。在`BufferPoolManager`中将`page_id`映射到`frame_id`时，STL是线程不安全的。

你需要实现以下函数：（头文件`src/include/buffer/buffer_pool_manager.h`，源文件`src/buffer/buffer_pool_manager.cpp`，测试文件`test/buffer/buffer_pool_manager_test.cpp`）

* `FetchPage(page_id_t page_id)`：若空闲列表中没有可用页面并且其他页面都`pinned`，则应返回`nullptr`。`FlushPage`应刷新页面，而不管其`pin`状态。
* `UnpinPage(page_id_t page_id, bool is_dirty)`：`is_dirty`参数跟踪页面在固定时是否被修改。
* `FlushPage(page_id_t page_id)`
* `NewPage(page_id_t* page_id)`
* `DeletePage(page_id_t page_id)`
* `FlushAllPages()`

当你要在`NewPage()`中创建新页面时，`AllocatePage`私有方法会提供`BufferPoolManager`唯一的新页面`ID`。Deallocate`Page()`方法是一种模拟释放磁盘页面的空操作。

## 阅读测试文件和头文件

<figure><img src="../.gitbook/assets/BufferPoolManager.png" alt=""><figcaption><p><code>buffer_pool_manager.h</code></p></figcaption></figure>

<figure><img src="../.gitbook/assets/DiskManager.png" alt=""><figcaption><p><code>disk_manager.h</code></p></figcaption></figure>

<figure><img src="../.gitbook/assets/Page.png" alt=""><figcaption><p><code>page.h</code></p></figcaption></figure>

```cpp
TEST(BufferPoolManagerTest, DISABLED_SampleTest) {
  const std::string db_name = "test.db";
  const size_t buffer_pool_size = 10;
  const size_t k = 5;

  auto *disk_manager = new DiskManager(db_name);
  auto *bpm = new BufferPoolManager(buffer_pool_size, disk_manager, k);

  page_id_t page_id_temp;
  auto *page0 = bpm->NewPage(&page_id_temp);

  // Scenario: The buffer pool is empty. We should be able to create a new page.
  ASSERT_NE(nullptr, page0);
  EXPECT_EQ(0, page_id_temp);

  // Scenario: Once we have a page, we should be able to read and write content.
  snprintf(page0->GetData(), BUSTUB_PAGE_SIZE, "Hello");
  EXPECT_EQ(0, strcmp(page0->GetData(), "Hello"));

  // Scenario: We should be able to create new pages until we fill up the buffer pool.
  for (size_t i = 1; i < buffer_pool_size; ++i) {
    EXPECT_NE(nullptr, bpm->NewPage(&page_id_temp));
  }

  // Scenario: Once the buffer pool is full, we should not be able to create any new pages.
  for (size_t i = buffer_pool_size; i < buffer_pool_size * 2; ++i) {
    EXPECT_EQ(nullptr, bpm->NewPage(&page_id_temp));
  }

  // Scenario: After unpinning pages {0, 1, 2, 3, 4} and pinning another 4 new pages,
  // there would still be one buffer page left for reading page 0.
  for (int i = 0; i < 5; ++i) {
    EXPECT_EQ(true, bpm->UnpinPage(i, true));
  }
  for (int i = 0; i < 4; ++i) {
    EXPECT_NE(nullptr, bpm->NewPage(&page_id_temp));
  }

  // Scenario: We should be able to fetch the data we wrote a while ago.
  page0 = bpm->FetchPage(0);
  EXPECT_EQ(0, strcmp(page0->GetData(), "Hello"));

  // Scenario: If we unpin page 0 and then make a new page, all the buffer pages should
  // now be pinned. Fetching page 0 should fail.
  EXPECT_EQ(true, bpm->UnpinPage(0, true));
  EXPECT_NE(nullptr, bpm->NewPage(&page_id_temp));
  EXPECT_EQ(nullptr, bpm->FetchPage(0));

  // Shutdown the disk manager and remove the temporary file we created.
  disk_manager->ShutDown();
  remove("test.db");

  delete bpm;
  delete disk_manager;
}
```

## NewPage(page\_id\_t \*page\_id)

```cpp
/**
 * TODO(P1): Add implementation
 *
 * @brief Create a new page in the buffer pool. Set page_id to the new page's id, or nullptr if all frames
 * are currently in use and not evictable (in another word, pinned).
 *
 * You should pick the replacement frame from either the free list or the replacer (always find from the free list
 * first), and then call the AllocatePage() method to get a new page id. If the replacement frame has a dirty page,
 * you should write it back to the disk first. You also need to reset the memory and metadata for the new page.
 *
 * Remember to "Pin" the frame by calling replacer.SetEvictable(frame_id, false)
 * so that the replacer wouldn't evict the frame before the buffer pool manager "Unpin"s it.
 * Also, remember to record the access history of the frame in the replacer for the lru-k algorithm to work.
 *
 * @param[out] page_id id of created page
 * @return nullptr if no new pages could be created, otherwise pointer to new page
 */
auto NewPage(page_id_t *page_id) -> Page *;
```

`@brief:`

* 从buffer pool中创建一个新page。为新page设置page\_id，或者如果其他帧都pinned返回nullptr。
* 你应该从free list或者replacer中选取replacement frame（优先free list），然后调用`AllocatePage()`获取一个新page\_id。若replacement frame有一个脏页，你应先将其写回disk。然后为新page重设memory和metadata。

`@param[out]:`

* `page_id`创建page的id

`@return：`

* 若无新页被创建，返回`nullptr`。否则返回指向新页的指针。

<figure><img src="broken-reference" alt=""><figcaption><p><code>NewPage</code></p></figcaption></figure>

