# Design Patterns

**Load**: `view .claude/skills/design-patterns/SKILL.md`

---

## 描述

常见设计模式及实用的 Java 示例。涵盖创建型、行为型和结构型模式，使用现代 Java 语法和 Spring 集成。

---

## 使用场景

- "Implement factory pattern for notifications"
- "Use builder for this complex object"
- "How to add features without modifying class?"（Decorator）
- "Multiple payment methods, swap at runtime"（Strategy）
- "Notify multiple services when order placed"（Observer）

---

## 示例

```
> view .claude/skills/design-patterns/SKILL.md
> "I need to create different report types (PDF, Excel, CSV)"
→ Suggests Factory pattern with implementation example
```

---

## 涵盖的模式

| Category | Patterns |
|----------|----------|
| **Creational** | Builder、Factory Method、Singleton |
| **Behavioral** | Strategy、Observer、Template Method |
| **Structural** | Decorator、Adapter |

---

## 快速选择指南

| Problem | Pattern |
|---------|---------|
| Many constructor parameters | Builder |
| Create without specifying class | Factory |
| Swap algorithms at runtime | Strategy |
| Add behavior dynamically | Decorator |
| Notify multiple objects | Observer |
| Integrate legacy code | Adapter |

---

## 相关技能

- `solid-principles` - 模式实现的原则
- `clean-code` - 代码级实践
- `spring-boot-patterns` - Spring 实现

---

## 参考资料

- [Refactoring Guru - Design Patterns](https://refactoring.guru/design-patterns)
- [Design Patterns by Gang of Four](https://www.oreilly.com/library/view/design-patterns-elements/0201633612/)
- [Java Design Patterns (java-design-patterns.com)](https://java-design-patterns.com/)
