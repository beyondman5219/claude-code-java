# API Contract Review Skill

> 审计 REST API 的 HTTP 语义、版本控制和一致性

## 功能描述

审查 REST API 设计：
- HTTP 动词正确性（GET vs POST vs PUT vs PATCH）
- API 版本控制策略
- 请求/响应结构（DTO vs entities）
- 状态码使用（不返回 200 状态码但包含错误响应体）
- 向后兼容性问题

## 何时使用

- "Review this API" / "Check REST endpoints"
- 发布 API 更改之前
- 审查 controller PR
- 检查 API 是否遵循 REST 最佳实践

## 核心概念

### Audit vs Template

| spring-boot-patterns | api-contract-review |
|---------------------|---------------------|
| 如何编写 controllers | 审查现有 API |
| 模板和示例 | 清单和反模式 |
| 创建新代码 | 审计现有代码 |

### 常见问题

| Issue | Example |
|-------|---------|
| 错误的动词 | 用 POST 进行搜索而非 GET |
| 没有版本控制 | `/users` 而不是 `/v1/users` |
| Entity 泄露 | 直接返回 JPA entity |
| 200 状态码但返回错误 | HTTP 200 + `{"status": "error"}` |
| 破坏性更改 | 请求中添加必填字段 |

## 使用示例

```
You: Review the API in UserController

Claude: [检查 HTTP 动词使用]
        [验证版本控制]
        [查找 entity 泄露]
        [审查错误处理]
        [识别破坏性更改]
```

## 检查内容

1. **HTTP Semantics** - 操作的正确动词
2. **URL Design** - 版本控制、命名约定
3. **Request Handling** - 验证、DTO
4. **Response Design** - DTO、分页、一致性
5. **Error Handling** - 状态码、错误格式
6. **Compatibility** - 破坏性 vs 非破坏性更改

## 相关技能

- `spring-boot-patterns` - 编写 controller 的模板（此技能审计它们）
- `security-audit` - API 的安全方面
- `java-code-review` - 通用代码审查（此技能专注于 API）

## 参考资料

- [REST API Design Best Practices](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [API Versioning](https://www.baeldung.com/rest-versioning)
