# 测试要求

## 最低测试覆盖率：80%

测试类型（全部必需）：

1. **单元测试** - 单个函数、工具、组件
2. **集成测试** - API 端点、数据库操作
3. **端到端测试** - 关键用户流程（根据语言选择框架）

## 测试驱动开发

强制工作流程：

1. 先写测试（红色）
2. 运行测试 - 它应该失败
3. 编写最小实现（绿色）
4. 运行测试 - 它应该通过
5. 重构（改进）
6. 验证覆盖率（80%+）

## 测试失败故障排除

1. 使用 **tdd-guide** 代理
2. 检查测试隔离性
3. 验证模拟是否正确
4. 修复实现，而不是测试（除非测试本身有误）

## 代理支持

* **tdd-guide** - 主动用于新功能，强制执行先写测试原则

## 测试结构（AAA 模式）

测试优先采用 Arrange-Act-Assert 结构：

```typescript
test('calculates similarity correctly', () => {
  // Arrange
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert
  expect(similarity).toBe(0)
})
```

### 测试命名

使用描述性名称来解释被测试的行为：

```typescript
test('returns empty array when no markets match query', () => {})
test('throws error when API key is missing', () => {})
test('falls back to substring search when Redis is unavailable', () => {})
```
