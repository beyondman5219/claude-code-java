---
name: logging-patterns
description: Java 日志最佳实践，使用 SLF4J、结构化日志（JSON）和 MDC 进行请求追踪。包含为 Claude Code 分析优化的 AI 友好日志格式。当用户询问日志、调试应用程序流程或分析日志时使用。
---

# Logging Patterns 技能

有效的 Java 应用程序日志，专注于结构化、AI 可解析的格式。

## 何时使用
- 用户说 "add logging" / "improve logs" / "debug this"
- 从日志分析应用程序流程
- 设置结构化日志（JSON）
- 使用 correlation IDs 进行请求追踪
- AI/Claude Code 需要分析应用程序行为

---

## AI 友好日志

> **关键洞察**：JSON 日志更适合 AI 分析 - 更快的解析、更少的 token、直接的字段访问。

### 为什么 AI/Claude Code 使用 JSON？

```
# Text format - AI 必须"解释"字符串
2026-01-29 10:15:30 INFO OrderService - Order 12345 created for user-789, total: 99.99

# JSON format - AI 直接提取字段
{"timestamp":"2026-01-29T10:15:30Z","level":"INFO","orderId":12345,"userId":"user-789","total":99.99}
```

| Aspect | Text | JSON |
|--------|------|------|
| 解析 | Regex/解释 | 直接字段访问 |
| Token 使用 | 更高（重复模式） | 更低（结构化） |
| 错误提取 | 解析堆栈跟踪文本 | `exception` 字段 |
| 过滤 | grep patterns | `jq` queries |

### AI 辅助开发的推荐设置

```yaml
# application.yml - 默认 JSON
logging:
  structured:
    format:
      console: logstash  # Spring Boot 3.4+

# 当你需要手动读取日志时：
# 选项 1：使用 jq
# tail -f app.log | jq .

# 选项 2：临时切换 profile
# java -jar app.jar --spring.profiles.active=human-logs
```

### 为 AI 分析优化的日志格式

```json
{
  "timestamp": "2026-01-29T10:15:30.123Z",
  "level": "INFO",
  "logger": "com.example.OrderService",
  "message": "Order created",
  "requestId": "req-abc123",
  "traceId": "trace-xyz",
  "orderId": 12345,
  "userId": "user-789",
  "duration_ms": 45,
  "step": "payment_completed"
}
```

**AI 调试的关键字段：**
- `requestId` - 将同一请求的所有日志分组
- `step` - 跟踪流程中的进度
- `duration_ms` - 识别慢操作
- `level` - 快速过滤错误

### 使用 AI/Claude Code 读取日志

当要求 AI 分析日志时：

```bash
# 获取最近的错误
cat app.log | jq 'select(.level == "ERROR")' | tail -20

# 跟踪特定请求
cat app.log | jq 'select(.requestId == "req-abc123")'

# 查找慢操作
cat app.log | jq 'select(.duration_ms > 1000)'
```

AI 可以：
1. 直接解析 JSON（无需猜测）
2. 通过 requestId 跟踪请求流程
3. 精确识别错误发生位置
4. 测量步骤之间的时间

---

## 快速设置（Spring Boot 3.4+）

### 原生结构化日志

Spring Boot 3.4+ 具有内置支持 - 无需额外依赖！

```yaml
# application.yml
logging:
  structured:
    format:
      console: logstash    # 或 "ecs" 用于 Elastic Common Schema

# 支持的格式：logstash、ecs、gelf
```

### 基于 Profile 的切换

```yaml
# application.yml（默认 - JSON 用于 AI/prod）
spring:
  profiles:
    default: json-logs

---
spring:
  config:
    activate:
      on-profile: json-logs
logging:
  structured:
    format:
      console: logstash

---
spring:
  config:
    activate:
      on-profile: human-logs
# 无结构化格式 = 人类可读的默认格式
logging:
  pattern:
    console: "%d{HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n"
```

**使用：**
```bash
# 默认：JSON（用于 AI、CI/CD、生产）
./mvnw spring-boot:run

# 需要时可读
./mvnw spring-boot:run -Dspring.profiles.active=human-logs
```

---

## Spring Boot < 3.4 的设置

### Logstash Logback Encoder

**pom.xml:**
```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

**logback-spring.xml:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- JSON（默认） -->
    <springProfile name="!human-logs">
        <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder">
                <includeMdcKeyName>requestId</includeMdcKeyName>
                <includeMdcKeyName>userId</includeMdcKeyName>
            </encoder>
        </appender>
        <root level="INFO">
            <appender-ref ref="JSON"/>
        </root>
    </springProfile>

    <!-- 人类可读（可选） -->
    <springProfile name="human-logs">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>

</configuration>
```

### 添加自定义字段（Logstash Encoder）

```java
import static net.logstash.logback.argument.StructuredArguments.kv;

// 字段显示为单独的 JSON 键
log.info("Order created",
    kv("orderId", order.getId()),
    kv("userId", user.getId()),
    kv("total", order.getTotal()),
    kv("step", "order_created")
);

// 输出：
// {"message":"Order created","orderId":123,"userId":"u-456","total":99.99,"step":"order_created"}
```

---

## SLF4J 基础

### Logger 声明

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class OrderService {
    private static final Logger log = LoggerFactory.getLogger(OrderService.class);
}

// 或使用 Lombok
@Slf4j
@Service
public class OrderService {
    // 直接使用 `log`
}
```

### 参数化日志

```java
// ✅ 好：仅在启用级别时评估
log.debug("Processing order {} for user {}", orderId, userId);

// ❌ 坏：总是拼接
log.debug("Processing order " + orderId + " for user " + userId);

// ✅ 对于昂贵的操作
if (log.isDebugEnabled()) {
    log.debug("Full order details: {}", order.toJson());
}
```

---

## 日志级别

| Level | 何时 | 示例 |
|-------|------|---------|
| **ERROR** | 需要注意的失败 | 未处理的异常、服务关闭 |
| **WARN** | 意外但已处理 | 重试成功、使用了已弃用的 API |
| **INFO** | 业务事件 | 订单创建、支付处理 |
| **DEBUG** | 技术细节 | 方法参数、SQL 查询 |
| **TRACE** | 非常详细 | 循环迭代（很少使用） |

```java
log.error("Payment failed", kv("orderId", id), kv("reason", reason), exception);
log.warn("Retry succeeded", kv("attempt", 3), kv("orderId", id));
log.info("Order shipped", kv("orderId", id), kv("trackingNumber", tracking));
log.debug("Fetching from DB", kv("query", "findById"), kv("id", id));
```

---

## MDC (Mapped Diagnostic Context)

MDC 为请求中的每个日志条目添加上下文 - 对追踪至关重要。

### Request ID Filter

```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class RequestContextFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {
        try {
            String requestId = Optional.ofNullable(request.getHeader("X-Request-ID"))
                .filter(s -> !s.isBlank())
                .orElse(UUID.randomUUID().toString().substring(0, 8));

            MDC.put("requestId", requestId);
            response.setHeader("X-Request-ID", requestId);

            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

### 添加用户上下文

```java
// 认证后
MDC.put("userId", authentication.getName());

// 所有后续日志自动包含 userId
log.info("User action performed");  // {"userId":"john123","message":"User action performed"}
```

### 异步操作中的 MDC

```java
// MDC 不会自动传播到新线程！

// ✅ 复制 MDC 上下文
Map<String, String> context = MDC.getCopyOfContextMap();

CompletableFuture.runAsync(() -> {
    try {
        if (context != null) MDC.setContextMap(context);
        log.info("Async task running");  // 有 requestId、userId
    } finally {
        MDC.clear();
    }
});
```

---

## 记录什么

### 业务事件（INFO）

```java
// 包含关键标识符和状态
log.info("Order created",
    kv("orderId", id),
    kv("userId", userId),
    kv("total", total),
    kv("itemCount", items.size()),
    kv("step", "order_created"));

log.info("Payment processed",
    kv("orderId", id),
    kv("amount", amount),
    kv("method", "card"),
    kv("step", "payment_completed"));
```

### 外部调用（带计时）

```java
long start = System.currentTimeMillis();
try {
    Result result = externalService.call(params);
    log.info("External call succeeded",
        kv("service", "PaymentGateway"),
        kv("operation", "charge"),
        kv("duration_ms", System.currentTimeMillis() - start));
    return result;
} catch (Exception e) {
    log.error("External call failed",
        kv("service", "PaymentGateway"),
        kv("operation", "charge"),
        kv("duration_ms", System.currentTimeMillis() - start),
        e);
    throw e;
}
```

### 流程步骤（用于 AI 追踪）

```java
public Order processOrder(CreateOrderRequest request) {
    log.info("Processing started", kv("step", "start"), kv("requestData", request.summary()));

    Order order = createOrder(request);
    log.info("Order created", kv("step", "order_created"), kv("orderId", order.getId()));

    validateInventory(order);
    log.info("Inventory validated", kv("step", "inventory_ok"), kv("orderId", order.getId()));

    processPayment(order);
    log.info("Payment processed", kv("step", "payment_done"), kv("orderId", order.getId()));

    log.info("Processing completed", kv("step", "complete"), kv("orderId", order.getId()));
    return order;
}
```

---

## 不记录什么

```java
// ❌ 永远不要记录敏感数据
log.info("Login", kv("password", password));           // 密码
log.info("Payment", kv("cardNumber", card));           // 完整卡号
log.info("Request", kv("token", jwtToken));            // Tokens
log.info("User", kv("ssn", socialSecurity));           // PII

// ✅ 安全的替代方案
log.info("Login attempted", kv("userId", userId));
log.info("Payment", kv("cardLast4", last4));
log.info("Token validated", kv("subject", sub), kv("exp", expiry));
```

---

## 异常日志

### 在边界记录一次

```java
// ❌ 坏：多次记录同一异常
void methodA() {
    try { methodB(); }
    catch (Exception e) { log.error("Error", e); throw e; }  // 日志 #1
}
void methodB() {
    try { methodC(); }
    catch (Exception e) { log.error("Error", e); throw e; }  // 日志 #2
}

// ✅ 好：仅在 service 边界记录
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> handle(Exception e, HttpServletRequest request) {
        log.error("Request failed",
            kv("path", request.getRequestURI()),
            kv("method", request.getMethod()),
            kv("errorType", e.getClass().getSimpleName()),
            e);  // 完整堆栈跟踪
        return ResponseEntity.status(500).body(errorResponse);
    }
}
```

### 包含上下文

```java
// ❌ 无用
log.error("Error occurred", e);

// ✅ 对调试有用
log.error("Order processing failed",
    kv("orderId", orderId),
    kv("step", "payment"),
    kv("userId", userId),
    kv("attemptNumber", attempt),
    e);
```

---

## 快速参考

```java
// === 设置 ===
private static final Logger log = LoggerFactory.getLogger(MyClass.class);

// === 带结构化字段的日志 ===
import static net.logstash.logback.argument.StructuredArguments.kv;

log.info("Event", kv("key1", value1), kv("key2", value2));
log.error("Failed", kv("context", ctx), exception);

// === MDC ===
MDC.put("requestId", requestId);
MDC.put("userId", userId);
// ... 所有日志现在包含这些
MDC.clear();  // 清理

// === 级别 ===
log.error()  // 失败
log.warn()   // 已处理的问题
log.info()   // 业务事件
log.debug()  // 技术细节
```

---

## 分析日志（AI/Human）

```bash
# 漂亮打印 JSON 日志
tail -f app.log | jq .

# 过滤错误
cat app.log | jq 'select(.level == "ERROR")'

# 跟踪请求流程
cat app.log | jq 'select(.requestId == "abc123")'

# 查找慢操作（>1s）
cat app.log | jq 'select(.duration_ms > 1000)'

# 获取步骤的时间线
cat app.log | jq 'select(.requestId == "abc123") | {time: .timestamp, step: .step, message: .message}'
```

---

## 相关技能

- `spring-boot-patterns` - Spring Boot 配置
- `jpa-patterns` - 数据库日志（SQL 查询）
- 未来：`observability-patterns` - 指标、追踪、完整可观测性
