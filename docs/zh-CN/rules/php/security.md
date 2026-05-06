---
paths:
  - "**/*.php"
  - "**/composer.lock"
  - "**/composer.json"
---

# PHP 安全

> 本文档在 [common/security.md](../common/security.md) 的基础上扩展了 PHP 相关的内容。

## 输入与输出

* 在框架边界验证请求输入（使用 `FormRequest`、Symfony Validator 或显式的 DTO 验证）。
* 在模板中默认转义输出；将原始 HTML 渲染视为必须说明理由的例外情况。
* 未经验证，切勿信任查询参数、Cookie、请求头或上传文件的元数据。

## 数据库安全

* 对所有动态查询使用预处理语句（`PDO`、Doctrine、Eloquent 查询构造器）。
* 避免在控制器/视图中拼接 SQL 字符串。
* 谨慎限定 ORM 的批量赋值范围，并明确允许写入的字段白名单。

## 密钥与依赖项

* 从环境变量或密钥管理器加载密钥，切勿从已提交的配置文件中读取。
* 在 CI 中运行 `composer audit`，并在添加新依赖项前审查其维护者的可信度。
* 有意识地锁定主版本号，并尽快移除已废弃的包。

## 认证与会话安全

* 使用 `password_hash()` / `password_verify()` 存储密码。
* 在身份验证和权限变更后重新生成会话标识符。
* 对会改变状态的 Web 请求强制实施 CSRF 保护。

## 参考

有关 Laravel 特定的安全指南，请参阅技能：`laravel-security`。
