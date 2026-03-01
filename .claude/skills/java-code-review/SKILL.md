---
name: java-code-review
description: Java 的系统化代码审查，包含空值安全、异常处理、并发和性能检查。当用户说 "review code"、"check this PR"、"code review" 或在合并更改之前使用。
---

# Java 代码审查技能

Java 项目的系统化代码审查清单。

## 何时使用
- 用户说 "review this code" / "check this PR" / "code review"
- 在合并 PR 之前
- 在实现功能之后

## 审查策略

1. **快速扫描** - 了解意图，识别范围
2. **清单检查** - 检查下面的每个类别
3. **总结** - 按严重性列出发现（严重 → 次要）

## 输出格式

```markdown
## 代码审查：[文件/功能名称]

### 严重
- [问题描述 + 行引用 + 建议]

### 改进
- [建议 + 理由]

### 次要/风格
- [吹毛求疵、可选改进]

### 观察到的良好实践
- [正反馈 - 对士气很重要]
```

---

## 审查清单

### 1. 空值安全

**检查：**
```java
// ❌ NPE 风险
String name = user.getName().toUpperCase();

// ✅ 安全
String name = Optional.ofNullable(user.getName())
    .map(String::toUpperCase)
    .orElse("");

// ✅ 也安全（提前返回）
if (user.getName() == null) {
    return "";
}
return user.getName().toUpperCase();
```

**标记：**
- 没有空值检查的链式方法调用
- 公共 API 上缺少 `@Nullable` / `@NonNull` 注解
- 没有 `isPresent()` 检查的 `Optional.get()`
- 从可能返回 `Optional` 或空集合的方法返回 `null`

**建议：**
- 对可能缺失的返回类型使用 `Optional`
- 对构造函数/方法参数使用 `Objects.requireNonNull()`
- 返回空集合而不是 null：`Collections.emptyList()`

### 2. 异常处理

**检查：**
```java
// ❌ 吞掉异常
try {
    process();
} catch (Exception e) {
    // 静默忽略
}

// ❌ 捕获过于宽泛
catch (Exception e) { }
catch (Throwable t) { }

// ❌ 丢失堆栈跟踪
catch (IOException e) {
    throw new RuntimeException(e.getMessage());
}

// ✅ 正确处理
catch (IOException e) {
    log.error("Failed to process file: {}", filename, e);
    throw new ProcessingException("File processing failed", e);
}
```

**标记：**
- 空的 catch 块
- 广泛捕获 `Exception` 或 `Throwable`
- 丢失原始异常（不链式）
- 使用异常进行流控制
- 通过 API 边界泄漏检查异常

**建议：**
- 使用上下文和堆栈跟踪记录日志
- 使用特定的异常类型
- 使用 `cause` 链式异常
- 考虑为域错误使用自定义异常

### 3. 集合与流

**检查：**
```java
// ❌ 迭代时修改
for (Item item : items) {
    if (item.isExpired()) {
        items.remove(item);  // ConcurrentModificationException
    }
}

// ✅ 使用 removeIf
items.removeIf(Item::isExpired);

// ❌ 用于简单操作的 Stream
list.stream().forEach(System.out::println);

// ✅ 简单循环更清晰
for (Item item : list) {
    System.out.println(item);
}

// ❌ 收集以修改
List<String> names = users.stream()
    .map(User::getName)
    .collect(Collectors.toList());
names.add("extra");  // 可能是不可变的！

// ✅ 显式可变列表
List<String> names = users.stream()
    .map(User::getName)
    .collect(Collectors.toCollection(ArrayList::new));
```

**标记：**
- 迭代期间修改集合
- 过度使用流进行简单操作
- 假设 `Collectors.toList()` 返回可变列表
- 不使用 `List.of()`、`Set.of()`、`Map.of()` 作为不可变集合
- 不理解含义的并行流

**建议：**
- `List.copyOf()` 用于防御性副本
- `removeIf()` 而不是迭代器删除
- 流用于转换，循环用于副作用

### 4. 并发

**检查：**
```java
// ❌ 非线程安全
private Map<String, User> cache = new HashMap<>();

// ✅ 线程安全
private Map<String, User> cache = new ConcurrentHashMap<>();

// ❌ Check-then-act 竞态条件
if (!map.containsKey(key)) {
    map.put(key, computeValue());
}

// ✅ 原子操作
map.computeIfAbsent(key, k -> computeValue());

// ❌ 双重检查锁定（没有 volatile 会破坏）
if (instance == null) {
    synchronized(this) {
        if (instance == null) {
            instance = new Instance();
        }
    }
}
```

**标记：**
- 没有同步的共享可变状态
- 没有原子性的 check-then-act 模式
- 共享变量上缺少 `volatile`
- 在非 final 对象上同步
- 非线程安全的延迟初始化

**建议：**
- 优先使用不可变对象
- 使用 `java.util.concurrent` 类
- `AtomicReference`、`AtomicInteger` 用于简单情况
- 考虑 `@ThreadSafe` / `@NotThreadSafe` 注解

### 5. Java 惯用语

**equals/hashCode：**
```java
// ❌ 只有 equals 没有 hashCode
@Override
public boolean equals(Object o) { ... }
// 缺少 hashCode！

// ❌ hashCode 中的可变字段
@Override
public int hashCode() {
    return Objects.hash(id, mutableField);  // 破坏 HashMap
}

// ✅ 使用不可变字段，同时实现两者
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof User user)) return false;
    return Objects.equals(id, user.id);
}

@Override
public int hashCode() {
    return Objects.hash(id);
}
```

**toString：**
```java
// ❌ 缺少 - 难以调试
// 没有 toString()

// ❌ 包含敏感数据
return "User{password='" + password + "'}";

// ✅ 对调试有用
@Override
public String toString() {
    return "User{id=" + id + ", name='" + name + "'}";
}
```

**Builders：**
```java
// ✅ 对于带有多个可选参数的类
User user = User.builder()
    .name("John")
    .email("john@example.com")
    .build();
```

**标记：**
- `equals` 没有 `hashCode`
- `hashCode` 中的可变字段
- 域对象上缺少 `toString`
- 带有 > 3-4 个参数的构造函数（建议使用 builder）
- 不使用 `instanceof` 模式匹配（Java 16+）

### 6. 资源管理

**检查：**
```java
// ❌ 资源泄漏
FileInputStream fis = new FileInputStream(file);
// ... 可能在关闭之前抛出

// ✅ Try-with-resources
try (FileInputStream fis = new FileInputStream(file)) {
    // ...
}

// ❌ 多个资源，错误顺序
try (BufferedWriter writer = new BufferedWriter(new FileWriter(file))) {
    // 如果 BufferedWriter 失败，FileWriter 可能不会关闭
}

// ✅ 分离声明
try (FileWriter fw = new FileWriter(file);
     BufferedWriter writer = new BufferedWriter(fw)) {
    // 两者都正确关闭
}
```

**标记：**
- 不对 `Closeable`/`AutoCloseable` 使用 try-with-resources
- 打开了资源但不在 try-with-resources 中
- 数据库连接/语句未正确关闭

### 7. API 设计

**检查：**
```java
// ❌ Boolean 参数
process(data, true, false);  // 这些是什么意思？

// ✅ 使用枚举或 builder
process(data, ProcessMode.ASYNC, ErrorHandling.STRICT);

// ❌ 为"未找到"返回 null
public User findById(Long id) {
    return users.get(id);  // 如果未找到则为 null
}

// ✅ 返回 Optional
public Optional<User> findById(Long id) {
    return Optional.ofNullable(users.get(id));
}

// ❌ 接受 null 集合
public void process(List<Item> items) {
    if (items == null) items = Collections.emptyList();
}

// ✅ 要求非 null，接受空
public void process(List<Item> items) {
    Objects.requireNonNull(items, "items must not be null");
}
```

**标记：**
- Boolean 参数（首选枚举）
- 带有 > 3 个参数的方法（考虑参数对象）
- 类似方法之间的不一致 null 处理
- 公共 API 输入上缺少验证

### 8. 性能考虑

**检查：**
```java
// ❌ 循环中的字符串拼接
String result = "";
for (String s : strings) {
    result += s;  // 每次迭代创建新的 String
}

// ✅ StringBuilder
StringBuilder sb = new StringBuilder();
for (String s : strings) {
    sb.append(s);
}

// ❌ 循环中的正则编译
for (String line : lines) {
    if (line.matches("pattern.*")) { }  // 每次编译正则
}

// ✅ 预编译模式
private static final Pattern PATTERN = Pattern.compile("pattern.*");
for (String line : lines) {
    if (PATTERN.matcher(line).matches()) { }
}

// ❌ 循环中的 N+1
for (User user : users) {
    List<Order> orders = orderRepo.findByUserId(user.getId());
}

// ✅ 批量获取
Map<Long, List<Order>> ordersByUser = orderRepo.findByUserIds(userIds);
```

**标记：**
- 循环中的字符串拼接
- 循环中的正则编译
- N+1 查询模式
- 在紧凑循环中创建可以重用的对象
- 不使用原始流（`IntStream`、`LongStream`）

### 9. 测试提示

**建议测试：**
- Null 输入
- 空集合
- 边界值
- 异常情况
- 并发访问（如适用）

---

## 严重性指南

| 严重性 | 标准 |
|----------|----------|
| **严重** | 安全漏洞、数据丢失风险、生产崩溃 |
| **高** | 可能的 Bug、重大性能问题、破坏 API 合约 |
| **中等** | 代码异味、可维护性问题、缺少最佳实践 |
| **低** | 风格、次要优化、建议 |

## Token 优化

- 专注于更改的行（使用 `git diff`）
- 不要重复明显的问题 - 分组类似的发现
- 引用行号，而不是完整的代码引用
- 跳过自动生成或测试夹具的文件

## 快速参考卡

| 类别 | 关键检查 |
|----------|------------|
| 空值安全 | 链式调用、Optional 误用、null 返回 |
| 异常 | 空 catch、广泛 catch、丢失堆栈跟踪 |
| 集合 | 迭代期间修改、stream vs loop |
| 并发 | 共享可变状态、check-then-act |
| 惯用语 | equals/hashCode 对、toString、builders |
| 资源 | try-with-resources、连接泄漏 |
| API | Boolean 参数、null 处理、验证 |
| 性能 | 字符串拼接、循环中的正则、N+1 |
