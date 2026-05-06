---
paths:
  - "**/*.cpp"
  - "**/*.hpp"
  - "**/*.cc"
  - "**/*.hh"
  - "**/*.cxx"
  - "**/*.h"
  - "**/CMakeLists.txt"
---

# C++ 模式

> 本文档扩展了 [common/patterns.md](../common/patterns.md) 的内容，增加了 C++ 特定部分。

## RAII（资源获取即初始化）

将资源生命周期与对象生命周期绑定：

```cpp
class FileHandle {
public:
    explicit FileHandle(const std::string& path) : file_(std::fopen(path.c_str(), "r")) {}
    ~FileHandle() { if (file_) std::fclose(file_); }
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
private:
    std::FILE* file_;
};
```

## 三五法则/零法则

* **零法则**：优先使用不需要自定义析构函数、拷贝/移动构造函数或赋值操作符的类
* **三五法则**：如果定义了析构函数、拷贝构造函数、拷贝赋值运算符、移动构造函数或移动赋值运算符中的任何一个，则应定义全部五个

## 值语义

* 小型/平凡类型按值传递
* 大型类型通过 `const&` 传递
* 按值返回（依赖 RVO/NRVO）
* 对接收参数使用移动语义

## 错误处理

* 对异常情况使用异常
* 对可能不存在的值使用 `std::optional`
* 对预期的失败使用 `std::expected`（C++23）或结果类型

## 参考

有关全面的 C++ 模式和反模式，请参阅技能：`cpp-coding-standards`。
