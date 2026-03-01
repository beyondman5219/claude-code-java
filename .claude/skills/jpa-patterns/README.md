# JPA Patterns

**Load**: `view .claude/skills/jpa-patterns/SKILL.md`

---

## 描述

Spring 应用程序的 JPA/Hibernate 模式和常见陷阱。涵盖 N+1 问题、lazy loading、事务、entity 关系和查询优化。

---

## 使用场景

- "Too many SQL queries executing"
- "LazyInitializationException error"
- "N+1 problem in my code"
- "How to optimize JPA queries?"
- "EAGER vs LAZY fetch type"
- "Entity relationship best practices"

---

## 示例

```
> view .claude/skills/jpa-patterns/SKILL.md
> "I see 100 queries when loading 10 orders"
→ Identifies N+1 problem, suggests JOIN FETCH or @EntityGraph
```

---

## 涵盖主题

| Topic | 关键点 |
|-------|------------|
| **N+1 Problem** | JOIN FETCH、@EntityGraph、@BatchSize |
| **Lazy Loading** | FetchType.LAZY、LazyInitializationException 解决方案 |
| **Transactions** | @Transactional、传播、只读 |
| **Relationships** | OneToMany、ManyToMany、双向同步 |
| **Optimization** | 分页、DTO 投影、批量操作 |
| **Locking** | @Version、OptimisticLockException |

---

## 常见错误处理

- @ManyToOne 上的 CascadeType.ALL
- 缺少数据库索引
- toString() 触发 lazy loading
- 从同一类调用 @Transactional

---

## 相关技能

- `spring-boot-patterns` - Spring Boot 模式
- `java-code-review` - 代码审查清单

---

## 参考资料

- [Hibernate ORM Documentation](https://hibernate.org/orm/documentation/)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
- [Vlad Mihalcea's Blog](https://vladmihalcea.com/) - JPA/Hibernate 深入分析
- [High-Performance Java Persistence](https://vladmihalcea.com/books/high-performance-java-persistence/)
