# 测试要求

## 最低测试覆盖率：80%

测试类型（全部必需）：
1. **单元测试** - 单个函数、工具、组件
2. **集成测试** - API 端点、数据库操作
3. **E2E 测试** - 关键用户流程（框架根据语言选择）

## 测试驱动开发

强制工作流：
1. 先写测试（RED）
2. 运行测试 - 应该失败
3. 编写最小实现（GREEN）
4. 运行测试 - 应该通过
5. 重构（IMPROVE）
6. 验证覆盖率（80%+）

## 测试失败排查

1. 使用 **tdd-guide** 代理
2. 检查测试隔离
3. 验证模拟是否正确
4. 修复实现，而非测试（除非测试有误）

## 代理支持

- **tdd-guide** - 主动用于新功能，强制先写测试

## 测试结构（AAA 模式）

测试优先使用 Arrange-Act-Assert 结构：

```typescript
test('正确计算相似度', () => {
  // Arrange（准备）
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act（执行）
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert（断言）
  expect(similarity).toBe(0)
})
```

### 测试命名

使用描述性名称解释被测试的行为：

```typescript
test('当查询无匹配市场时返回空数组', () => {})
test('当 API 密钥缺失时抛出错误', () => {})
test('当 Redis 不可用时回退到子字符串搜索', () => {})
```
