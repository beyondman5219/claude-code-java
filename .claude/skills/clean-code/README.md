# Clean Code

**加载**：`view .claude/skills/clean-code/SKILL.md`

---

## 描述

Clean Code 原则及 Java 示例：DRY、KISS、YAGNI、命名约定、函数设计、代码异味和重构技术。

---

## 使用场景

- "清理这段代码"
- "重构这个方法"
- "提高可读性"
- "这个函数太长了"
- "我应该如何命名这个变量？"
- "这段代码太复杂了吗？"

---

## 示例

```
> view .claude/skills/clean-code/SKILL.md
> "这个方法有 100 行，帮我重构"
→ 识别代码异味，建议使用 Extract Method、Guard Clauses
```

---

## 涵盖的原则

| 原则 | 关键问题 |
|-----------|--------------|
| **DRY** | 这段逻辑是否在其他地方重复了？ |
| **KISS** | 有更简单的方法吗？ |
| **YAGNI** | 我们现在需要这个，还是"以防万一"？ |

---

## 主题

- 命名约定（变量、方法、类）
- 函数设计（大小、参数、抽象）
- 注释（何时好，何时坏）
- 代码异味检测
- 重构技术

---

## 相关技能

- `solid-principles` - 类设计原则
- `design-patterns` - 常见解决方案
- `java-code-review` - 完整审查清单

---

## 参考资料

- [Robert C. Martin 的 Clean Code](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [Martin Fowler 的 Refactoring](https://refactoring.com/)
- [Refactoring Guru - 代码异味](https://refactoring.guru/refactoring/smells)
