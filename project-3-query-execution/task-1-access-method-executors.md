---
description: 访问方法执行程序
---

# task#1 Access Method Executors

## 背景：查询处理

BusTub的架构如下：

<figure><img src="../.gitbook/assets/bustub_arch.svg" alt=""><figcaption><p>BusTub's architecture</p></figcaption></figure>

在公共BusTub数据库中，我们提供了一个完整的查询处理层。你可以使用BusTub shell来执行SQL查询，就像在其他数据库系统中一样。使用以下命令编译并运行BusTub shell:

```sh
cd build && make -j$(nproc) shell
./bin/bustub-shell
```

你还可以使用[`BusTub Web Shell`](https://15445.courses.cs.cmu.edu/spring2023/bustub/)运行以下示例。它是在浏览器中运行的系统的完整参考解决方案。

在shell中，可用`\dt`查看所有表。
