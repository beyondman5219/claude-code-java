# Performance Smell Detection Skill

> 识别潜在的代码级性能问题 - 带有细微差别，而非绝对标准

## 功能描述

帮助发现 Java 代码中的**潜在**性能异味：
- Stream API 使用模式
- 装箱/拆箱开销
- Regex 编译成本
- 集合低效
- 字符串操作

**理念**："先测量，后优化" - 现代 JVM 已经高度优化。

## 何时使用

- "Check for performance issues"
- "Review this hot path"
- "Is this code efficient?"
- 调查测量到的缓慢

## Java 版本意识

此技能考虑了现代 Java 优化：

| Topic | Java 9+ 变化 |
|-------|----------------|
| String `+` | 使用 invokedynamic，循环外已优化良好 |
| StringBuilder | 循环中仍然最好 |
| Virtual Threads | Java 21+ 用于 I/O 密集型工作 |
| String hashCode | Java 25 常量折叠 |

## 严重性级别

| Level | 含义 | 操作 |
|-------|---------|--------|
| 🔴 High | 通常值得修复 | 主动修复 |
| 🟡 Medium | 先测量 | 更改前进行分析 |
| 🟢 Low | 最好有 | 仅在关键路径上 |

## 检查内容

1. **Strings** - 循环中的拼接（仍然是合理关注点）
2. **Streams** - 紧密循环中的开销、并行误用
3. **Boxing** - 热路径中的基本类型包装器
4. **Regex** - 循环中的 Pattern.compile
5. **Collections** - 错误类型、无界查询
6. **Modern patterns** - Virtual threads、structured concurrency

## 不检查的内容

- **JPA/Database** - 使用 `jpa-patterns` skill
- **Architecture** - 使用 `architecture-review` skill
- **JVM tuning** - 超出范围（GC、heap 等）

## 使用示例

```
You: Check this code for performance issues

Claude: [识别潜在异味]
        [评估严重性：🔴/🟡/🟢]
        [建议更改前先测量]
        [如适用，建议现代替代方案]
```

## 相关技能

- `jpa-patterns` - 数据库性能（N+1、分页）
- `java-code-review` - 通用代码质量
- `concurrency-review` - 线程安全和异步模式

## 参考资料

- [Inside.java - JDK 25 Performance](https://inside.java/2025/10/20/jdk-25-performance-improvements/)
- [Java 25 Features - InfoQ](https://www.infoq.com/news/2025/09/java25-released/)
- [Baeldung - Streams vs Loops](https://www.baeldung.com/java-streams-vs-loops)
- [Baeldung - String Concatenation](https://www.baeldung.com/java-string-concatenation-methods)
