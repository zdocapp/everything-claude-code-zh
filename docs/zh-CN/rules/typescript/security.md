---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---

# TypeScript/JavaScript 安全

> 本文档在 [common/security.md](../common/security.md) 的基础上扩展了 TypeScript/JavaScript 相关的内容。

## 密钥管理

```typescript
// NEVER: Hardcoded secrets
const apiKey = "sk-proj-xxxxx"

// ALWAYS: Environment variables
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

## 代理支持

* 使用 **security-reviewer** 技能进行全面的安全审计
