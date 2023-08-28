# task#4 SQL String Functions

## 要求

现在是时候深入了解BusTub本身了！你将要实现上层和下层SQL函数。这可以分两步完成：

* 在`string_expression.h`中实现函数逻辑
* 在BusTub中注册函数，让SQL框架在用户执行一条SQL时调用你的函数（`plan_func_call.cpp`）。

你可以使用`bustub-shell`测试你的实现：

```sh
cd build
make -j`nproc` shell
./bin/bustub-shell
bustub> select upper('AbCd'), lower('AbCd');
ABCD abcd
```

你的实现应该通过3条sqllogictest测试案例：

```sh
cd build
make -j`nproc` sqllogictest
./bin/bustub-sqllogictest ../test/sql/p0.01-lower-upper.slt --verbose
./bin/bustub-sqllogictest ../test/sql/p0.02-function-error.slt --verbose
./bin/bustub-sqllogictest ../test/sql/p0.03-string-scan.slt --verbose
```

