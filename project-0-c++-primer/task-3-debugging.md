# task#3 Debugging

## 要求

在本task中，你将学习调试的基本知识。你可以选择任何你喜欢的调试方式，包括但不限于：使用cout和printf，使用CLion/VScode，gdb等。

有关说明，请阅读`trie_debug.cpp`。生成trie后需要设置断点并回答几个问题。你需要在`trie_answer.h`中填写答案。

## 阅读`trie_debug.cpp`

```cpp
TEST(TrieDebugger, TestCase) {
  std::mt19937_64 gen(2333);
  std::uniform_int_distribution<uint32_t> dis(0, 100);

  auto trie = Trie();
  for (uint32_t i = 0; i < 10; i++) {
    std::string key = fmt::format("{}", dis(gen));
    auto value = dis(gen);
    trie = trie.Put<uint32_t>(key, value);
  }

  // Put a breakpoint here.

  // (1) How many children nodes are there on the root?
  // Replace `CASE_1_YOUR_ANSWER` in `trie_answer.h` with the correct answer.
  if (CASE_1_YOUR_ANSWER != Case1CorrectAnswer()) {
    ASSERT_TRUE(false);
  }

  // (2) How many children nodes are there on the node of prefix `9`?
  // Replace `CASE_2_YOUR_ANSWER` in `trie_answer.h` with the correct answer.
  if (CASE_2_YOUR_ANSWER != Case2CorrectAnswer()) {
    ASSERT_TRUE(false);
  }

  // (3) What's the value for `93`?
  // Replace `CASE_3_YOUR_ANSWER` in `trie_answer.h` with the correct answer.
  if (CASE_3_YOUR_ANSWER != Case3CorrectAnswer()) {
    ASSERT_TRUE(false);
  }
}
```

## 步骤

在指定处设置断点进行调试，得到三个参数填入`trie_answer.h`。
