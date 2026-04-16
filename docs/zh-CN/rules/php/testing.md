---
paths:
  - "**/*.php"
  - "**/phpunit.xml"
  - "**/phpunit.xml.dist"
  - "**/composer.json"
---

# PHP 测试

> 本文档在 [common/testing.md](../common/testing.md) 的基础上扩展了 PHP 相关的内容。

## 框架

使用 **PHPUnit** 作为默认测试框架。如果项目中配置了 **Pest**，则新测试应优先使用 Pest，并避免混合使用不同框架。

## 覆盖率

```bash
vendor/bin/phpunit --coverage-text
# or
vendor/bin/pest --coverage
```

在 CI 环境中优先使用 **pcov** 或 **Xdebug**，并将覆盖率阈值设置在 CI 中，而非作为团队内部知识。

## 测试组织

* 将快速的单元测试与框架/数据库集成测试分开。
* 使用工厂/构建器来创建测试数据，而非手动编写大型数组。
* 保持 HTTP/控制器测试专注于传输和验证；将业务规则移至服务层测试。

## Inertia

如果项目使用 Inertia.js，优先使用 `assertInertia` 配合 `AssertableInertia` 来验证组件名称和属性，而非使用原始的 JSON 断言。

## 参考

查看技能：`tdd-workflow` 了解整个代码库的 RED -> GREEN -> REFACTOR 循环。
查看技能：`laravel-tdd` 了解 Laravel 特定的测试模式（PHPUnit 和 Pest）。
