# Spring Boot Patterns

**Load**: `view .claude/skills/spring-boot-patterns/SKILL.md`

---

## 描述

Spring Boot 应用程序的最佳实践和模式。涵盖项目结构、分层架构、DTO、异常处理、配置和测试。

---

## 使用场景

- "Create a REST controller for products"
- "Add service layer for user management"
- "Setup global exception handling"
- "How should I structure this Spring Boot project?"

---

## 示例

```
> view .claude/skills/spring-boot-patterns/SKILL.md
> "Create UserController with CRUD endpoints"
→ Generates controller following REST conventions with proper status codes
```

---

## 涵盖的模式

| Layer | Topics |
|-------|--------|
| Controller | REST 约定、验证、状态码 |
| Service | Interface + Impl、事务、mappers |
| Repository | JPA 查询、派生方法、优化 |
| DTO | Request/Response records、MapStruct |
| Exception | 自定义异常、全局处理器 |
| Config | Properties、profiles、验证 |
| Testing | MockMvc、Mockito、Testcontainers |

---

## 注意事项 / 提示

- 使用构造器注入（Lombok `@RequiredArgsConstructor`）
- 默认在 service 类级别使用 `@Transactional(readOnly = true)`
- 永远不要直接暴露 entity - 使用 DTO
- DTO 优先使用 records（Java 17+）
