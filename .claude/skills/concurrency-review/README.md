# Concurrency Review Skill

> 审查 Java 并发代码的线程安全性、竞态条件和现代模式

## 功能描述

审查多线程 Java 代码：
- 竞态条件和可见性问题
- 死锁可能性
- 现代模式（Virtual Threads、Structured Concurrency）
- Spring @Async 陷阱
- CompletableFuture 错误处理
- 线程池配置

## 为什么重要

> 近 60% 的多线程应用程序由于共享资源管理不当而遇到问题。

并发 bugs 难以复现、难以测试、难以调试。在代码审查中捕获它们比在生产中发现它们要好得多。

## 何时使用

- "Review this for thread safety"
- "Check concurrency issues"
- "Is this async code correct?"
- 审查包含 `synchronized`、`volatile`、`@Async` 的代码
- 检查 `CompletableFuture` 或 `ExecutorService` 使用

## 涵盖的核心主题

### Modern Java (21/25)
| Topic | 检查内容 |
|-------|---------------|
| Virtual Threads | 用于 I/O 密集型，而非 CPU 密集型 |
| Structured Concurrency | 正确的作用域管理 |
| ScopedValue | 优于 ThreadLocal |

### Spring @Async
| Pitfall | Issue |
|---------|-------|
| 同类调用 | 绕过代理，同步运行 |
| 非公共方法 | 代理无法拦截 |
| 默认 executor | 每个任务创建线程（OOM 风险） |
| SecurityContext | ThreadLocal 不会传播 |

### Classic Issues
| Issue | Example |
|-------|---------|
| 竞态条件 | 没有同步的 check-then-act |
| 可见性 | 缺少 volatile |
| 死锁 | 不一致的锁顺序 |

## 使用示例

```
You: Review this service for thread safety

Claude: [检查共享可变状态]
        [验证同步]
        [审查 @Async 配置]
        [检查 CompletableFuture 错误处理]
        [如适用，建议现代替代方案]
```

## 严重性级别

| Level | 含义 |
|-------|---------|
| 🔴 High | 可能是 bug - 竞态条件、死锁风险 |
| 🟡 Medium | 潜在问题 - 需要测量/验证 |
| 🟢 Modern | Java 21/25 模式的机会 |

## 相关技能

- `performance-smell-detection` - 性能问题（非线程安全）
- `java-code-review` - 通用代码审查（包括基础并发）
- `spring-boot-patterns` - Spring 模式（包括 @Async 基础）

## 参考资料

- [Java Concurrency Code Review Checklist](https://github.com/code-review-checklists/java-concurrency)
- [Baeldung - Common Concurrency Pitfalls](https://www.baeldung.com/java-common-concurrency-pitfalls)
- [Oracle - Virtual Threads](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html)
- [JavaPro - Java 25 Virtual Threads](https://javapro.io/2025/12/23/java-25-getting-the-most-out-of-virtual-threads-with-structured-task-scopes-and-scoped-values/)
- [Spring @Async Problems](https://serdaralkancode.medium.com/problems-and-solutions-when-using-async-in-spring-boot-e383f9d3b45d)
- Book: "Java Concurrency in Practice" by Brian Goetz
