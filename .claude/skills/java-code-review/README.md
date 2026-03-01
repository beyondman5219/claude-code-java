# Java 代码审查

**加载**：`view .claude/skills/java-code-review/SKILL.md`

---

## 描述

Java 项目的系统化代码审查清单。涵盖空值安全、异常处理、集合、并发、惯用语、资源管理、API 设计和性能。

---

## 使用场景

- "审查这个类"
- "检查这个 PR 的问题"
- "审查 PluginManager 中的更改"
- "这段代码有什么问题？"

---

## 示例

```
> view .claude/skills/java-code-review/SKILL.md
> "审查 src/main/java/org/example/UserService.java 中的更改"
→ 按严重性分组返回发现（严重 → 次要）
```

---

## 清单类别

1. **空值安全** - NPE 风险、Optional 使用
2. **异常处理** - 吞掉的异常、堆栈跟踪
3. **集合与流** - 迭代、可变性
4. **并发** - 线程安全、竞态条件
5. **Java 惯用语** - equals/hashCode、builders
6. **资源管理** - try-with-resources
7. **API 设计** - Boolean 参数、验证
8. **性能** - 字符串拼接、N+1 查询

---

## 注意事项 / 提示

- 最适合专注于更改（单个类或 PR）
- 包含良好实践的正反馈部分
- 建议为审查期间发现的边缘情况添加测试
