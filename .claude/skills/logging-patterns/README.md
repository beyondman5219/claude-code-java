# Logging Patterns

**Load**: `view .claude/skills/logging-patterns/SKILL.md`

---

## 描述

Java 日志最佳实践，使用 SLF4J、结构化日志（JSON）和 MDC 进行请求追踪。包含为 Claude Code 分析优化的 AI 友好日志格式。

---

## 使用场景

- "Add logging to this service"
- "Debug this flow"（AI 读取日志）
- "Setup structured logging"
- "Why is this request failing?"（分析日志）
- "Add request tracing"

---

## 核心洞察：JSON for AI

**JSON 日志更适合 AI/Claude Code 分析：**

| Aspect | Text Logs | JSON Logs |
|--------|-----------|-----------|
| 解析 | Regex 解释 | 直接字段访问 |
| Token | 更高 | 更低 |
| 过滤 | grep patterns | jq queries |

```bash
# AI 可以轻松过滤 JSON
cat app.log | jq 'select(.requestId == "abc123")'
```

---

## 涵盖主题

| Topic | 描述 |
|-------|-------------|
| **AI-Friendly Logging** | 为 Claude Code 优化的 JSON 格式 |
| **Spring Boot 3.4+** | 原生结构化日志支持 |
| **Logstash Encoder** | 适用于 Spring Boot < 3.4 |
| **SLF4J/MDC** | 请求上下文、correlation IDs |
| **Log Levels** | 何时使用 ERROR、WARN、INFO、DEBUG |
| **What to Log** | 业务事件、计时、流程步骤 |
| **What NOT to Log** | 密码、PII、敏感数据 |

---

## 快速设置（Spring Boot 3.4+）

```yaml
logging:
  structured:
    format:
      console: logstash
```

不需要额外的依赖！

---

## 相关技能

- `spring-boot-patterns` - Spring 配置
- `jpa-patterns` - 数据库日志

---

## 参考资料

- [Structured Logging in Spring Boot 3.4 (spring.io)](https://spring.io/blog/2024/08/23/structured-logging-in-spring-boot-3-4/)
- [Structured Logging in Spring Boot (Baeldung)](https://www.baeldung.com/spring-boot-structured-logging)
- [10 Best Practices for Logging in Java (Better Stack)](https://betterstack.com/community/guides/logging/how-to-start-logging-with-java/)
- [Booking.com - Structured Logging](https://medium.com/booking-com-development/unlocking-observability-structured-logging-in-spring-boot-c81dbabfb9e7)
- [SLF4J Manual](https://www.slf4j.org/manual.html)
