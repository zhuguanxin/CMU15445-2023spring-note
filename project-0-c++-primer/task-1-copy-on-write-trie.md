# task#1 copy on write Trie

_In this task, you will need to modify `trie.h` and `trie.cpp` to implement a copy-on-write trie. In a copy-on-write trie, operations do not directly modify the nodes of the original trie. Instead, new nodes are created for modified data, and a new root is returned for the newly-modified trie. Copy-on-write enables us to access the trie after each operation at any time with minimum overhead. Consider inserting `("ad", 2)` in the above example. We create a new `Node2` by reusing two of the child nodes from the original tree, and creating a new value node 2. (See figure below)_

在
