---
name: clean-code
description: Clean Code 原则（DRY、KISS、YAGNI）、命名约定、函数设计和重构。当用户说 "clean this code"、"refactor"、"improve readability" 或审查代码质量时使用。
---

# Clean Code 技能

遵循 Clean Code 原则编写可读、可维护的代码。

## 何时使用
- 用户说 "clean this code" / "refactor" / "improve readability"
- 专注于可维护性的代码审查
- 降低复杂性
- 改进命名

---

## 核心原则

| 原则 | 含义 | 违反标志 |
|-----------|---------|----------------|
| **DRY** | 不要重复自己 | 复制粘贴的代码块 |
| **KISS** | 保持简单，傻瓜 | 过度设计的解决方案 |
| **YAGNI** | 你不会需要它 | "以防万一"的功能 |

---

## DRY - 不要重复自己

> "系统中的每一条知识都必须有一个单一、明确的表示。"

### 违反

```java
// ❌ 坏：相同的验证逻辑重复
public class UserController {

    public void createUser(UserRequest request) {
        if (request.getEmail() == null || request.getEmail().isBlank()) {
            throw new ValidationException("Email is required");
        }
        if (!request.getEmail().contains("@")) {
            throw new ValidationException("Invalid email format");
        }
        // ... 创建用户
    }

    public void updateUser(UserRequest request) {
        if (request.getEmail() == null || request.getEmail().isBlank()) {
            throw new ValidationException("Email is required");
        }
        if (!request.getEmail().contains("@")) {
            throw new ValidationException("Invalid email format");
        }
        // ... 更新用户
    }
}
```

### 重构后

```java
// ✅ 好：单一真实来源
public class EmailValidator {

    public void validate(String email) {
        if (email == null || email.isBlank()) {
            throw new ValidationException("Email is required");
        }
        if (!email.contains("@")) {
            throw new ValidationException("Invalid email format");
        }
    }
}

public class UserController {
    private final EmailValidator emailValidator;

    public void createUser(UserRequest request) {
        emailValidator.validate(request.getEmail());
        // ... 创建用户
    }

    public void updateUser(UserRequest request) {
        emailValidator.validate(request.getEmail());
        // ... 更新用户
    }
}
```

### DRY 例外

并非所有重复都是坏事。避免过早抽象：

```java
// 这些看起来相似但服务于不同的目的 - 可以重复
public BigDecimal calculateShippingCost(Order order) {
    return order.getWeight().multiply(SHIPPING_RATE);
}

public BigDecimal calculateInsuranceCost(Order order) {
    return order.getValue().multiply(INSURANCE_RATE);
}
// 不要把这些强行放在一个方法中 - 它们会有不同的发展
```

---

## KISS - 保持简单

> "最简单的解决方案通常是最好的。"

### 违反

```java
// ❌ 坏：针对简单任务过度设计
public class StringUtils {

    public boolean isEmpty(String str) {
        return Optional.ofNullable(str)
            .map(String::trim)
            .map(String::isEmpty)
            .orElseGet(() -> Boolean.TRUE);
    }
}
```

### 重构后

```java
// ✅ 好：简单清晰
public class StringUtils {

    public boolean isEmpty(String str) {
        return str == null || str.trim().isEmpty();
    }

    // 或使用现有库
    // return StringUtils.isBlank(str);  // Apache Commons
    // return str == null || str.isBlank();  // Java 11+
}
```

### KISS 清单

- 初级开发者能在 30 秒内理解这个吗？
- 有使用标准库的更简单方法吗？
- 我是否为可能永远不会发生的边缘情况增加了复杂性？

---

## YAGNI - 你不会需要它

> "在必要之前不要添加功能。"

### 违反

```java
// ❌ 坏：为假设的未来构建
public interface Repository<T, ID> {
    T findById(ID id);
    List<T> findAll();
    List<T> findAll(Pageable pageable);
    List<T> findAll(Sort sort);
    List<T> findAllById(Iterable<ID> ids);
    T save(T entity);
    List<T> saveAll(Iterable<T> entities);
    void delete(T entity);
    void deleteById(ID id);
    void deleteAll(Iterable<T> entities);
    void deleteAll();
    boolean existsById(ID id);
    long count();
    // ... 20 多个方法"以防万一"
}

// 当前使用：只有 findById 和 save
```

### 重构后

```java
// ✅ 好：只有现在需要的
public interface UserRepository {
    Optional<User> findById(Long id);
    User save(User user);
}

// 在实际需要时添加方法，而不是之前
```

### YAGNI 迹象

- "我们以后可能需要这个"
- "让我们让它可配置以防万一"
- "如果我们将来需要支持 X 怎么办？"
- 只有一个实现的抽象类

---

## 命名约定

### 变量

```java
// ❌ 坏
int d;                  // d 是什么？
String s;               // 无意义
List<User> list;        // 什么类型的列表？
Map<String, Object> m;  // 它映射什么？

// ✅ 好
int elapsedTimeInDays;
String customerName;
List<User> activeUsers;
Map<String, Object> sessionAttributes;
```

### 布尔值

```java
// ❌ 坏
boolean flag;
boolean status;
boolean check;

// ✅ 好 - 使用 is/has/can/should 前缀
boolean isActive;
boolean hasPermission;
boolean canEdit;
boolean shouldNotify;
```

### 方法

```java
// ❌ 坏
void process();           // 处理什么？
void handle();            // 处理什么？
void doIt();              // 做什么？
User get();               // 从哪里获取？

// ✅ 好 - 动词 + 名词，描述性
void processPayment();
void handleLoginRequest();
void sendWelcomeEmail();
User findByEmail(String email);
List<Order> fetchPendingOrders();
```

### 类

```java
// ❌ 坏
class Data { }           // 太模糊
class Info { }           // 太模糊
class Manager { }        // 通常是上帝类
class Helper { }         // 通常是垃圾场
class Utils { }          // 静态方法垃圾场

// ✅ 好 - 名词，特定职责
class User { }
class OrderProcessor { }
class EmailValidator { }
class PaymentGateway { }
class ShippingCalculator { }
```

### 命名约定表

| 元素 | 约定 | 示例 |
|---------|------------|---------|
| 类 | PascalCase，名词 | `OrderService` |
| 接口 | PascalCase，形容词或名词 | `Comparable`、`List` |
| 方法 | camelCase，动词 | `calculateTotal()` |
| 变量 | camelCase，名词 | `customerEmail` |
| 常量 | UPPER_SNAKE | `MAX_RETRY_COUNT` |
| 包 | 小写 | `com.example.orders` |

---

## 函数 / 方法

### 保持函数小

```java
// ❌ 坏：50+ 行的方法做多件事
public void processOrder(Order order) {
    // 验证订单（10 行）
    // 计算总计（15 行）
    // 应用折扣（10 行）
    // 更新库存（10 行）
    // 发送通知（10 行）
    // ... 以及更多
}

// ✅ 好：小而专注的方法
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotals(order);
    applyDiscounts(order);
    updateInventory(order);
    sendNotifications(order);
}
```

### 单一抽象层次

```java
// ❌ 坏：混合抽象层次
public void processOrder(Order order) {
    validateOrder(order);  // 高层次

    // 低层次混合进来
    BigDecimal total = BigDecimal.ZERO;
    for (OrderItem item : order.getItems()) {
        total = total.add(item.getPrice().multiply(
            BigDecimal.valueOf(item.getQuantity())));
    }

    sendEmail(order);  // 又是高层次
}

// ✅ 好：一致的抽象层次
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    sendConfirmation(order);
}

private BigDecimal calculateTotal(Order order) {
    return order.getItems().stream()
        .map(item -> item.getPrice().multiply(
            BigDecimal.valueOf(item.getQuantity())))
        .reduce(BigDecimal.ZERO, BigDecimal::add);
}
```

### 限制参数

```java
// ❌ 坏：参数太多
public User createUser(String firstName, String lastName,
                       String email, String phone,
                       String address, String city,
                       String country, String zipCode) {
    // ...
}

// ✅ 好：使用参数对象
public User createUser(CreateUserRequest request) {
    // ...
}

// 或使用 builder
public User createUser(UserBuilder builder) {
    // ...
}
```

### 避免标志参数

```java
// ❌ 坏：Boolean 标志改变行为
public void sendMessage(String message, boolean isUrgent) {
    if (isUrgent) {
        // 立即发送
    } else {
        // 稍后排队
    }
}

// ✅ 好：单独的方法
public void sendUrgentMessage(String message) {
    // 立即发送
}

public void queueMessage(String message) {
    // 稍后排队
}
```

---

## 注释

### 避免明显的注释

```java
// ❌ 坏：噪音注释
// 设置用户名
user.setName(name);

// 增加计数器
counter++;

// 检查用户是否为 null
if (user != null) {
    // ...
}
```

### 好的注释

```java
// ✅ 好：解释 WHY，而不是 WHAT

// 使用指数退避重试，以免在高负载期间使服务器不堪重负
//（参见事件 #1234）
for (int attempt = 0; attempt < MAX_RETRIES; attempt++) {
    Thread.sleep((long) Math.pow(2, attempt) * 1000);
    // ...
}

// TODO：在基础设施升级后用 Redis 缓存替换（2026 年第二季度）
private Map<String, User> userCache = new ConcurrentHashMap<>();

// 警告：顺序很重要！折扣必须在税计算之前应用
applyDiscounts(order);
calculateTax(order);
```

### 让代码说话

```java
// ❌ 坏：注释解释糟糕的代码
// 检查用户是否是管理员或有特殊权限
// 以及该操作是否被其角色允许
if ((user.getRole() == 1 || user.getRole() == 2) &&
    (action == 3 || action == 4 || action == 7)) {
    // ...
}

// ✅ 好：自文档化代码
if (user.hasAdminPrivileges() && action.isAllowedFor(user.getRole())) {
    // ...
}
```

---

## 常见代码异味

| 异味 | 描述 | 重构 |
|-------|-------------|-------------|
| **长方法** | 方法 > 20 行 | 提取方法 |
| **长参数列表** | > 3 个参数 | 参数对象 |
| **重复代码** | 多处相同代码 | 提取方法/类 |
| **死代码** | 未使用的代码 | 删除它 |
| **魔法数字** | 无法解释的字面量 | 命名常量 |
| **上帝类** | 类做得太多 | 提取类 |
| **功能嫉妒** | 方法使用另一个类的数据 | 移动方法 |
| **基本类型偏执** | 基本类型而不是对象 | 值对象 |

### 魔法数字

```java
// ❌ 坏
if (user.getAge() >= 18) { }
if (order.getTotal() > 100) { }
Thread.sleep(86400000);

// ✅ 好
private static final int ADULT_AGE = 18;
private static final BigDecimal FREE_SHIPPING_THRESHOLD = new BigDecimal("100");
private static final long ONE_DAY_MS = TimeUnit.DAYS.toMillis(1);

if (user.getAge() >= ADULT_AGE) { }
if (order.getTotal().compareTo(FREE_SHIPPING_THRESHOLD) > 0) { }
Thread.sleep(ONE_DAY_MS);
```

### 基本类型偏执

```java
// ❌ 坏：到处都是基本类型
public void createUser(String email, String phone, String zipCode) {
    // 没有验证，容易混淆参数
}

createUser("12345", "john@email.com", "555-1234");  // 错误顺序，编译通过！

// ✅ 好：值对象
public record Email(String value) {
    public Email {
        if (!value.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
    }
}

public record PhoneNumber(String value) {
    // 验证
}

public void createUser(Email email, PhoneNumber phone, ZipCode zipCode) {
    // 类型安全，自我验证
}
```

---

## 重构快速参考

| 从 | 到 | 技术 |
|------|-----|-----------|
| 长方法 | 短方法 | 提取方法 |
| 重复代码 | 单个方法 | 提取方法 |
| 复杂条件 | 多态 | 用多态替换条件 |
| 许多参数 | 对象 | 引入参数对象 |
| 临时变量 | 查询方法 | 用查询替换临时变量 |
| 解释代码的注释 | 自文档化代码 | 重命名、提取 |
| 嵌套条件 | 提前返回 | 保护子句 |

### 保护子句

```java
// ❌ 坏：深度嵌套
public void processOrder(Order order) {
    if (order != null) {
        if (order.isValid()) {
            if (order.hasItems()) {
                // 实际逻辑埋在这里
            }
        }
    }
}

// ✅ 好：保护子句
public void processOrder(Order order) {
    if (order == null) return;
    if (!order.isValid()) return;
    if (!order.hasItems()) return;

    // 实际逻辑在顶层
}
```

---

## Clean Code 清单

审查代码时，检查：

- [ ] 名称有意义且可发音吗？
- [ ] 函数小而专注吗？
- [ ] 有重复的代码吗？
- [ ] 有魔法数字或字符串吗？
- [ ] 注释解释的是"为什么"而不是"什么"吗？
- [ ] 代码是否在一致的抽象层次？
- [ ] 有任何代码可以简化吗？
- [ ] 有死代码/未使用的代码吗？

---

## 相关技能

- `solid-principles` - 类结构的设计原则
- `design-patterns` - 重复问题的常见解决方案
- `java-code-review` - 综合审查清单
