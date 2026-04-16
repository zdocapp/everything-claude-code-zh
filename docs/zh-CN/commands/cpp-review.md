---
description: 全面的C++代码审查，涵盖内存安全、现代C++惯用法、并发性和安全性。调用cpp-reviewer代理。
---

# C++ 代码审查

此命令调用 **cpp-reviewer** 代理进行全面的 C++ 专项代码审查。

## 此命令的功能

1. **识别 C++ 变更**：通过 `git diff` 查找修改过的 `.cpp`、`.hpp`、`.cc`、`.h` 文件
2. **运行静态分析**：执行 `clang-tidy` 和 `cppcheck`
3. **内存安全检查**：检查原始 new/delete、缓冲区溢出、释放后使用
4. **并发审查**：分析线程安全性、互斥锁使用、数据竞争
5. **现代 C++ 检查**：验证代码是否遵循 C++17/20 规范和最佳实践
6. **生成报告**：按严重程度对问题进行归类

## 使用时机

在以下情况使用 `/cpp-review`：

* 编写或修改 C++ 代码后
* 提交 C++ 变更前
* 审查包含 C++ 代码的拉取请求时
* 接手新的 C++ 代码库时
* 检查内存安全问题

## 审查类别

### 严重（必须修复）

* 未使用 RAII 的原始 `new`/`delete`
* 缓冲区溢出和释放后使用
* 无同步的数据竞争
* 通过 `system()` 导致的命令注入
* 读取未初始化的变量
* 空指针解引用

### 高（应该修复）

* 五法则违规
* 缺少 `std::lock_guard` / `std::scoped_lock`
* 分离的线程没有正确的生命周期管理
* 使用 C 风格强制转换而非 `static_cast`/`dynamic_cast`
* 缺少 `const` 正确性

### 中（考虑修复）

* 不必要的拷贝（按值传递而非 `const&`）
* 已知大小的容器缺少 `reserve()`
* 头文件中的 `using namespace std;`
* 重要的返回值缺少 `[[nodiscard]]`
* 过于复杂的模板元编程

## 运行的自动化检查

```bash
# Static analysis
clang-tidy --checks='*,-llvmlibc-*' src/*.cpp -- -std=c++17

# Additional analysis
cppcheck --enable=all --suppress=missingIncludeSystem src/

# Build with warnings
cmake --build build -- -Wall -Wextra -Wpedantic
```

## 使用示例

````text
User: /cpp-review

Agent:
# C++ 代码审查报告

## 已审查文件
- src/handler/user.cpp (已修改)
- src/service/auth.cpp (已修改)

## 静态分析结果
✓ clang-tidy: 2 个警告
✓ cppcheck: 无问题

## 发现的问题

[严重] 内存泄漏
文件: src/service/auth.cpp:45
问题: 使用了原始的 `new` 而没有匹配的 `delete`
```cpp
auto* session = new Session(userId);  // 内存泄漏！
cache[userId] = session;
````

修复：使用 `std::unique_ptr`

```cpp
auto session = std::make_unique<Session>(userId);
cache[userId] = std::move(session);
```

\[高] 缺少常量引用
文件：src/handler/user.cpp:28
问题：大型对象按值传递

```cpp
void processUser(User user) {  // Unnecessary copy
```

修复：通过常量引用传递

```cpp
void processUser(const User& user) {
```

## 总结

* 严重：1
* 高：1
* 中：0

建议：失败：在严重问题修复前阻止合并

```

## 批准标准

| 状态 | 条件 |
|--------|-----------|
| PASS: 批准 | 无 CRITICAL 或 HIGH 级别问题 |
| WARNING: 警告 | 仅存在 MEDIUM 级别问题（谨慎合并） |
| FAIL: 阻止 | 发现 CRITICAL 或 HIGH 级别问题 |

## 与其他命令的集成

- 首先使用 `/cpp-test` 以确保测试通过
- 如果出现构建错误，请使用 `/cpp-build`
- 在提交前使用 `/cpp-review`
- 对于非 C++ 特定问题，请使用 `/code-review`

## 相关

- Agent: `agents/cpp-reviewer.md`
- Skills: `skills/cpp-coding-standards/`, `skills/cpp-testing/`
```
