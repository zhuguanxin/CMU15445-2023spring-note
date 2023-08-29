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

## 步骤

```cpp
// plan_func_call.cpp
auto Planner::GetFuncCallFromFactory(const std::string &func_name, std::vector<AbstractExpressionRef> args)
    -> AbstractExpressionRef {
  // 1. check if the parsed function name is "lower" or "upper".
  // 2. verify the number of args (should be 1), refer to the test cases for when you should throw an `Excepetion`.
  // 3. return a `StringExpression` std::shared_ptr.
  throw Exception(fmt::format("func call {} not supported in planner yet", func_name));
}
```

* 检查解析函数的名字是否"lower"或"upper"
* 检查参数的数字（应该为1），当你抛出一个"Excepetion"时，指向test cases
* 返回一个`` `StringExpression` std::shared_ptr ``

首先在`string_expression.h`中实现`Compute(val)`函数。通过成员变量`StringExpressionType`判断`val`需要大写`toupper`还是小写`tolower`。

然后在`plan_func_call.cpp`实现`GetFuncCallFromFactory(func_name,args)`函数。
