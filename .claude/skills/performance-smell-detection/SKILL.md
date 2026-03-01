---
name: performance-smell-detection
description: 识别 Java 代码中潜在的代码级性能问题 - streams、collections、boxing、regex、对象创建。提供意识，而非绝对标准 - 优化前先测量。对于 JPA/数据库性能，请改用 jpa-patterns。
---

# Performance Smell Detection 技能

识别 Java 代码中**潜在的**代码级性能问题。

## 理念

> "Premature optimization is the root of all evil" - Donald Knuth

此技能帮助你**注意**潜在的性能异味，而不是盲目地"修复"它们。现代 JVM（Java 21/25）已高度优化。始终：

1. **先测量** - 使用 JMH、profilers 或生产指标
2. **专注于热路径** - 90% 的时间花在 10% 的代码上
3. **考虑可读性** - 清晰的代码通常比微优化更重要

## 何时使用
- 审查性能关键代码路径
- 调查测量到的性能问题
- 学习 Java 性能模式
- 带性能意识的代码审查

## 范围

**此技能**：代码级性能（streams、collections、对象）
**数据库**：使用 `jpa-patterns` 技能（N+1、lazy loading、pagination）
**架构**：使用 `architecture-review` 技能

---

## 快速参考：潜在异味

| 异味 | 严重性 | 上下文 |
|-------|----------|---------|
| 循环中 Regex compile | 🔴 High | 始终值得修复 |
| 循环中 String concat | 🟡 Medium | 在 Java 21/25 中仍然有效 |
| 紧密循环中的 Stream | 🟡 Medium | 取决于集合大小 |
| 热路径中的 Boxing | 🟡 Medium | 先测量 |
| 无界集合 | 🔴 High | 内存风险 |
| 缺少集合容量 | 🟢 Low | 次要的，如果关键则测量 |

---

## 字符串操作（Java 9+ / 21 / 25）

### 有什么变化

自 **Java 9**（JEP 280）起，使用 `+` 的字符串拼接使用 `invokedynamic`，而非 StringBuilder。JVM 很好地优化了简单拼接。

**Java 25** 添加了 String::hashCode 常量折叠，为使用 String 键的 Map 查找提供了额外的优化。

### 仍然有效：循环中的 StringBuilder

```java
// 🔴 仍有问题 - 每次迭代创建新 String
String result = "";
for (String s : items) {
    result += s;  // O(n²) - 创建 n 个 strings
}

// ✅ 循环使用 StringBuilder
StringBuilder sb = new StringBuilder();
for (String s : items) {
    sb.append(s);
}
String result = sb.toString();

// ✅ 或使用 String.join / Collectors.joining
String result = String.join("", items);
```

### 现在可以：简单拼接

```java
// ✅ 在 Java 9+ 中可以 - JVM 优化了这个
String message = "User " + name + " logged in at " + timestamp;

// ✅ 也可以
return "Error: " + code + " - " + description;
```

### 热路径中避免：String.format

```java
// 🟡 String.format 有解析开销
log.debug(String.format("Processing %s with id %d", name, id));

// ✅ 参数化日志（SLF4J）
log.debug("Processing {} with id {}", name, id);
```

---

## Stream API（细致观点）

### 现实

Streams 有开销，但**通常可以接受**：
- **< 100 项**：Streams 可能慢 2-5 倍（但仍然是微秒级）
- **1K-10K 项**：差异显著缩小
- **> 10K 项**：通常在循环的 50% 以内
- **GraalVM**：可以优化 streams 以匹配循环

**推荐**：为了可读性优先使用 streams。仅当 profiling 显示瓶颈时才优化为循环。

### Streams 有问题的情况

```java
// 🔴 在热循环中每次迭代创建 Stream
for (int i = 0; i < 1_000_000; i++) {
    boolean found = items.stream()
        .anyMatch(item -> item.getId() == i);
}

// ✅ 预计算查找结构
Set<Integer> itemIds = items.stream()
    .map(Item::getId)
    .collect(Collectors.toSet());

for (int i = 0; i < 1_000_000; i++) {
    boolean found = itemIds.contains(i);
}
```

### Streams 可以的情况

```java
// ✅ 单次传递、可读、不在紧密循环中
List<String> names = users.stream()
    .filter(User::isActive)
    .map(User::getName)
    .sorted()
    .collect(Collectors.toList());

// ✅ 基本类型 streams 避免 boxing
int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();
```

### Parallel Streams：谨慎使用

```java
// 🔴 在小集合上并行 - 开销 > 收益
smallList.parallelStream().map(...);  // < 10K 项

// 🔴 使用共享可变状态的并行
List<String> results = new ArrayList<>();
items.parallelStream()
    .forEach(results::add);  // 竞态条件！

// ✅ 适用于 CPU 密集型 + 大集合的并行
List<Result> results = largeDataset.parallelStream()  // > 10K 项
    .map(this::expensiveCpuComputation)
    .collect(Collectors.toList());
```

---

## Boxing/Unboxing

### 仍然是真正的问题

Boxing 在堆上创建对象，增加 GC 压力。JVM 缓存小值（-128 到 127）但不缓存较大的值。

> **未来**：Project Valhalla 将显著改善这一点。

```java
// 🔴 紧密循环中的 Boxing - 创建数百万个对象
Long sum = 0L;
for (int i = 0; i < 1_000_000; i++) {
    sum += i;  // Unbox、add、box
}

// ✅ 基本类型
long sum = 0L;
for (int i = 0; i < 1_000_000; i++) {
    sum += i;
}
```

### 使用基本类型 Streams

```java
// 🟡 Boxing 开销
int sum = list.stream()
    .reduce(0, Integer::sum);

// ✅ 基本类型 stream
int sum = list.stream()
    .mapToInt(Integer::intValue)
    .sum();
```

---

## Regex

### 始终在循环中预编译

此建议**没有过时** - Pattern.compile 是昂贵的。

```java
// 🔴 每次迭代编译 pattern
for (String input : inputs) {
    if (input.matches("\\d{3}-\\d{4}")) {  // 编译 regex！
        process(input);
    }
}

// ✅ 预编译
private static final Pattern PHONE = Pattern.compile("\\d{3}-\\d{4}");

for (String input : inputs) {
    if (PHONE.matcher(input).matches()) {
        process(input);
    }
}
```

---

## Collections

### 容量提示（次要优化）

```java
// 🟢 低严重性 - 但如果大小已知则是免费优化
List<User> users = new ArrayList<>(expectedSize);
Map<String, User> map = new HashMap<>(expectedSize * 4 / 3 + 1);
```

### 为任务选择正确的集合

```java
// 🟡 循环中的 O(n) 查找
List<String> allowed = getAllowed();
for (Request r : requests) {
    if (allowed.contains(r.getId())) { }  // 每次 O(n)
}

// ✅ O(1) 查找
Set<String> allowed = new HashSet<>(getAllowed());
for (Request r : requests) {
    if (allowed.contains(r.getId())) { }  // O(1)
}
```

### 无界集合

```java
// 🔴 内存风险 - 可能无界增长
@GetMapping("/users")
public List<User> getAllUsers() {
    return userRepository.findAll();  // 数百万行？
}

// ✅ 分页
@GetMapping("/users")
public Page<User> getUsers(Pageable pageable) {
    return userRepository.findAll(pageable);
}
```

---

## 现代 Java (21/25) 模式

### Virtual Threads 用于 I/O（Java 21+）

```java
// 🟡 用于 I/O 的传统线程池 - 浪费 OS 线程
ExecutorService executor = Executors.newFixedThreadPool(100);
for (Request request : requests) {
    executor.submit(() -> callExternalApi(request));  // 阻塞 OS 线程
}

// ✅ Virtual threads - 数百万并发 I/O 操作
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (Request request : requests) {
        executor.submit(() -> callExternalApi(request));
    }
}
```

### Structured Concurrency（Java 21+ Preview）

```java
// ✅ 用于并行 I/O 的结构化并发
try (StructuredTaskScope.ShutdownOnFailure scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<User> user = scope.fork(() -> fetchUser(id));
    Future<Orders> orders = scope.fork(() -> fetchOrders(id));

    scope.join();
    scope.throwIfFailed();

    return new UserProfile(user.resultNow(), orders.resultNow());
}
```

---

## 性能审查清单

### 🔴 高严重性（通常值得修复）
- [ ] 循环中的 Regex Pattern.compile
- [ ] 没有分页的无界查询
- [ ] 循环中的字符串拼接（StringBuilder 仍然有效）
- [ ] 使用共享可变状态的并行 streams

### 🟡 中等严重性（先测量）
- [ ] 紧密循环中的 Streams（>100K 次迭代）
- [ ] 热路径中的 Boxing
- [ ] 循环中的 List.contains()（使用 Set）
- [ ] 用于 I/O 的传统线程（考虑 Virtual Threads）

### 🟢 低严重性（最好有）
- [ ] 集合初始容量
- [ ] 次要的 stream 优化
- [ ] toArray(new T[0]) vs toArray(new T[size])

---

## 何时不优化

- **不是热路径** - 设置代码、配置、管理 endpoints
- **没有测量到的问题** - "看起来慢"不是测量
- **可读性受损** - 清晰的代码 > 微优化
- **小集合** - 100 项在任何情况下都在微秒内处理

---

## 分析命令

```bash
# 查找循环中的 regex（潜在的编译开销）
grep -rn "\.matches(\|\.split(" --include="*.java"

# 查找潜在的 boxing（Long/Integer 作为变量）
grep -rn "Long\s\|Integer\s\|Double\s" --include="*.java" | grep "= 0\|+="

# 查找没有容量的 ArrayList
grep -rn "new ArrayList<>()" --include="*.java"

# 查找没有分页的 findAll
grep -rn "findAll()" --include="*.java"
```
