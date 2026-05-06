---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---

# Perl 测试

> 本文档在 [common/testing.md](../common/testing.md) 的基础上扩展了 Perl 相关的内容。

## 测试框架

对于新项目，请使用 **Test2::V0**（而非 Test::More）：

```perl
use Test2::V0;

is($result, 42, 'answer is correct');

done_testing;
```

## 测试运行器

```bash
prove -l t/              # adds lib/ to @INC
prove -lr -j8 t/         # recursive, 8 parallel jobs
```

务必使用 `-l` 以确保 `lib/` 位于 `@INC` 上。

## 测试覆盖率

使用 **Devel::Cover** —— 覆盖率目标为 80% 以上：

```bash
cover -test
```

## 模拟

* **Test::MockModule** —— 模拟现有模块的方法
* **Test::MockObject** —— 从头创建测试替身

## 常见陷阱

* 测试文件末尾务必加上 `done_testing`
* 使用 `prove` 时，切勿忘记 `-l` 标志

## 参考

有关使用 Test2::V0、prove 和 Devel::Cover 的详细 Perl TDD 模式，请参阅技能：`perl-testing`。
