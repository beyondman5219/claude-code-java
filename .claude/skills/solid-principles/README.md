# SOLID Principles

**Load**: `view .claude/skills/solid-principles/SKILL.md`

---

## 描述

SOLID 原则清单，包含详细的 Java 示例。每个原则包括违反示例、重构解决方案和检测模式。

---

## 使用场景

- "Check this class for SOLID violations"
- "Is this class doing too much?"（SRP）
- "How do I add new types without modifying code?"（OCP）
- "Why shouldn't Square extend Rectangle?"（LSP）
- "This interface is too big"（ISP）
- "How to make this testable?"（DIP）

---

## 示例

```
> view .claude/skills/solid-principles/SKILL.md
> "Review this UserService for SOLID principles"
→ Identifies SRP violation, suggests extraction of validation and notification
```

---

## 涵盖的原则

| Principle | 关键问题 |
|-----------|--------------|
| **S**ingle Responsibility | 它是否只有一个变更原因？ |
| **O**pen/Closed | 我是否可以在不修改的情况下扩展？ |
| **L**iskov Substitution | 子类型是否可以替换基类型？ |
| **I**nterface Segregation | 客户端是否被迫实现未使用的方法？ |
| **D**ependency Inversion | 它是否依赖于抽象？ |

---

## 相关技能

- `design-patterns` - 实现模式
- `clean-code` - DRY、KISS、YAGNI
- `java-code-review` - 完整审查清单

---

## 参考资料

- [SOLID (Wikipedia)](https://en.wikipedia.org/wiki/SOLID)
- [Clean Code by Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [SOLID Principles in Java (Baeldung)](https://www.baeldung.com/solid-principles)
