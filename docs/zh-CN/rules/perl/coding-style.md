---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---

# Perl 编码风格

> 本文档在 [common/coding-style.md](../common/coding-style.md) 的基础上扩展了 Perl 相关的内容。

## 标准

* 始终 `use v5.36`（启用 `strict`、`warnings`、`say`、子程序签名）
* 使用子程序签名 —— 切勿手动解包 `@_`
* 优先使用 `say` 而非显式换行的 `print`

## 不可变性

* 对所有属性使用带有 `is => 'ro'` 和 `Types::Standard` 的 **Moo**
* 切勿直接使用受祝福的哈希引用 —— 始终使用 Moo/Moose 访问器
* **面向对象覆盖说明**：对于计算出的只读值，使用带有 `builder` 或 `default` 的 Moo `has` 属性是可接受的

## 格式化

使用 **perltidy** 并应用以下设置：

```
-i=4    # 4 空格缩进
-l=100  # 100 字符行宽
-ce     # else 紧贴前括号
-bar    # 左大括号始终在右侧
```

## 代码检查

使用 **perlcritic**，严重级别设为 3，主题包括：`core`、`pbp`、`security`。

```bash
perlcritic --severity 3 --theme 'core || pbp || security' lib/
```

## 参考

有关全面的现代 Perl 惯用法和最佳实践，请参阅技能：`perl-patterns`。
