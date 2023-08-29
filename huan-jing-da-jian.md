# 环境搭建

* 课程官网：[https://15445.courses.cs.cmu.edu/spring2023/](https://15445.courses.cs.cmu.edu/spring2023/)
* 课程GitHub：[https://github.com/cmu-db/bustub](https://github.com/cmu-db/bustub)
* 参考书目：[数据库系统概念（第七版）](https://www.db-book.com/)
* OS：Ubuntu 22.04.2(WSL2)
* IDE：VScode

## 拉取代码

```sh
mkdir bustub2023spring
cd bustub2023spring
git clone git@github.com:cmu-db/bustub.git
cd bustub/
git checkout -b branchname v20230514-2023spring
```

## 检查基本依赖包

<pre class="language-sh"><code class="lang-sh">clang-tidy --version
<strong>#Ubuntu LLVM version 14.0.0
</strong>#  
#  Optimized build.
#  Default target: x86_64-pc-linux-gnu
<strong>#  Host CPU: alderlake
</strong></code></pre>

```sh
clang-format --version
#Ubuntu clang-format version 14.0.0-1ubuntu1
```

## 编译代码

```sh
mkdir build
cd build
cmake ..
make trie_test
#出现编译进程说明顺利编译
./test/trie_test
```

## 格式化代码

```sh
make format
make check-lint
make check-clang-tidy-p0
```

## 压缩包

```sh
#以project0为例
make submit-p0
```

在文件中找到`project0-submission.zip`上传。
