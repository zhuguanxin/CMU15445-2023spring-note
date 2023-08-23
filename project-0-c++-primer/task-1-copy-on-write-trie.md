# task#1 Copy on Write Trie

## 字典树

字典树`Trie`是一种利用给定`key`检索`value`的有序树数据结构。为了简化，我们假定`key`都是可变长度字符串（但实际上可以是任何类型）。

`Trie`中每个节点可以有多个子节点，表示不同的下一个字符。



<figure><img src="../.gitbook/assets/trie-01.svg" alt=""><figcaption><p>key "ab"的value 1存储在左子节点，key "ac"的value val存储在右子节点</p></figcaption></figure>

## 要求

在本task中，你需要修改`trie.h`和`trie.cpp`来完成一个写时复制字典树。在写时复制字典树中，操作不会直接修改原始trie的节点，而是为修改后的数据创建新节点，返回新的root。



<figure><img src="../.gitbook/assets/trie-02.svg" alt=""><figcaption><p>插入("ad",2)，重用原字典树的子节点并创建新子节点2</p></figcaption></figure>



<figure><img src="../.gitbook/assets/trie-03.svg" alt=""><figcaption><p>插入("b",3)</p></figcaption></figure>



<figure><img src="../.gitbook/assets/trie-04.svg" alt=""><figcaption><p>插入("a",abc)并删除("ab",1)</p></figcaption></figure>

你的trie应支持三个操作：

* `Get(key)`：获取key对应的value。
* `Put(key,value)`：为key设置指定的value。若key已存在，则覆盖原value。注意，value的类型可能是不可复制的(`std::unique_ptr<int>`)。返回一个新trie。
* `Remove(key)`：删除key的value。返回一个新trie。（此处官网有误，以bustub代码为准）

你应创建新的trie节点并尽可能重用原有节点。

若要创建新节点，应在`TrieNode`类上使用`Clone`函数。若要重用原有节点，你可以复制`std::shared_ptr<TrieNode>`（复制一个智能指针而不是复制指向的data）。不应使用`new`和`delete`分配内存。

## Put(key,value)

首先阅读测试用例`/test/primer/trie_test.cpp`。

```cpp
TEST(TrieTest, BasicPutTest) {
  auto trie = Trie();
  trie = trie.Put<uint32_t>("test-int", 233);
  trie = trie.Put<uint64_t>("test-int2", 23333333);
  trie = trie.Put<std::string>("test-string", "test");
  trie = trie.Put<std::string>("", "empty-key");
}
```

