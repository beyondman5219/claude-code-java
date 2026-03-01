---
name: java-migration
description: 在主要 LTS 版本之间升级 Java 项目的分步指南（8→11→17→21→25）。当用户说 "upgrade Java"、"migrate to Java 25"、"update Java version" 或现代化遗留项目时使用。
---

# Java Migration 技能

在主要版本之间升级 Java 项目的分步指南。

## 何时使用
- 用户说 "upgrade to Java 25" / "migrate from Java 8" / "update Java version"
- 现代化遗留项目
- Spring Boot 2.x → 3.x → 4.x 迁移
- 准备采用 LTS 版本

## 迁移路径

```
Java 8 (LTS) → Java 11 (LTS) → Java 17 (LTS) → Java 21 (LTS) → Java 25 (LTS)
     │              │               │              │               │
     └──────────────┴───────────────┴──────────────┴───────────────┘
                         始终迁移 LTS → LTS
```

---

## 快速参考：什么会破坏

| From → To | 主要破坏性更改 |
|-----------|------------------------|
| 8 → 11 | 删除了 `javax.xml.bind`、模块系统、内部 API |
| 11 → 17 | Sealed classes（preview→final）、强封装 |
| 17 → 21 | Pattern matching 更改、`finalize()` 弃用并计划删除 |
| 21 → 25 | 删除 Security Manager、删除 Unsafe 方法、放弃 32 位支持 |

---

## 迁移工作流

### 步骤 1：评估当前状态

```bash
# 检查当前 Java 版本
java -version

# 检查 Maven 中的编译器目标
grep -r "maven.compiler" pom.xml

# 查找已删除 API 的使用
grep -r "sun\." --include="*.java" src/
grep -r "javax\.xml\.bind" --include="*.java" src/
```

### 步骤 2：更新构建配置

**Maven:**
```xml
<properties>
    <java.version>21</java.version>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
</properties>

<!-- 或使用编译器插件 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.12.1</version>
    <configuration>
        <release>21</release>
    </configuration>
</plugin>
```

**Gradle:**
```groovy
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}
```

### 步骤 3：修复编译错误

运行编译并迭代修复错误：
```bash
mvn clean compile 2>&1 | head -50
```

### 步骤 4：运行测试

```bash
mvn test
```

### 步骤 5：检查运行时警告

```bash
# 使用 illegal-access 警告运行
java --illegal-access=warn -jar app.jar
```

---

## Java 8 → 11 迁移

### 删除的 API

| 删除的 | 替代品 |
|---------|-------------|
| `javax.xml.bind` (JAXB) | 添加依赖：`jakarta.xml.bind-api` + `jaxb-runtime` |
| `javax.activation` | 添加依赖：`jakarta.activation-api` |
| `javax.annotation` | 添加依赖：`jakarta.annotation-api` |
| `java.corba` | 无替代品（很少使用） |
| `java.transaction` | 添加依赖：`jakarta.transaction-api` |
| `sun.misc.Base64*` | 使用 `java.util.Base64` |
| `sun.misc.Unsafe`（部分）| 尽可能使用 `VarHandle` |

### 添加缺失的依赖（Maven）

```xml
<!-- JAXB（如果需要）-->
<dependency>
    <groupId>jakarta.xml.bind</groupId>
    <artifactId>jakarta.xml.bind-api</artifactId>
    <version>4.0.1</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
    <version>4.0.4</version>
    <scope>runtime</scope>
</dependency>

<!-- Annotation API -->
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>
```

### 模块系统问题

如果对 JDK 内部使用反射，添加 JVM 标志：
```bash
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/java.util=ALL-UNNAMED
```

**Maven Surefire:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <argLine>
            --add-opens java.base/java.lang=ALL-UNNAMED
        </argLine>
    </configuration>
</plugin>
```

### 采用的新功能

```java
// var（局部变量类型推断）
var list = new ArrayList<String>();  // 而不是 ArrayList<String> list = ...

// String 方法
"  hello  ".isBlank();      // 对于仅空白字符为 true
"  hello  ".strip();        // 更好的 trim()（Unicode 感知）
"line1\nline2".lines();     // Stream<String>
"ha".repeat(3);             // "hahaha"

// 集合工厂方法（Java 9+）
List.of("a", "b", "c");     // 不可变 list
Set.of(1, 2, 3);            // 不可变 set
Map.of("k1", "v1");         // 不可变 map

// Optional 改进
optional.ifPresentOrElse(
    value -> process(value),
    () -> handleEmpty()
);

// HTTP Client（替换 HttpURLConnection）
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com"))
    .build();
HttpResponse<String> response = client.send(request,
    HttpResponse.BodyHandlers.ofString());
```

---

## Java 11 → 17 迁移

### 破坏性更改

| Change | Impact |
|--------|--------|
| 强封装 | `--illegal-access` 不再起作用，必须使用显式 `--add-opens` |
| Sealed classes（final）| 如果你使用了 preview 功能 |
| Pattern matching instanceof | Preview → 最终语法更改 |

### 采用的新功能

```java
// Records（不可变数据类）
public record User(String name, String email) {}
// 自动生成：构造器、getter、equals、hashCode、toString

// Sealed classes
public sealed class Shape permits Circle, Rectangle {}
public final class Circle extends Shape {}
public final class Rectangle extends Shape {}

// instanceof 的 Pattern matching
if (obj instanceof String s) {
    System.out.println(s.length());  // s 已经转换
}

// Switch 表达式
String result = switch (day) {
    case MONDAY, FRIDAY -> "Work";
    case SATURDAY, SUNDAY -> "Rest";
    default -> "Midweek";
};

// Text blocks
String json = """
    {
        "name": "John",
        "age": 30
    }
    """;

// 有用的 NullPointerException 消息
// a.b.c.d() → 精确告诉你哪部分为 null
```

---

## Java 17 → 21 迁移

### 破坏性更改

| Change | Impact |
|--------|--------|
| Pattern matching switch（final）| 与 preview 有轻微语法差异 |
| `finalize()` 弃用并计划删除 | 替换为 `Cleaner` 或 try-with-resources |
| 默认 UTF-8 | 如果假设平台编码，可能影响文件读取 |

### 采用的新功能

```java
// Virtual Threads（Project Loom）- 主要功能
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> handleRequest());
}
// 或者简单地：
Thread.startVirtualThread(() -> doWork());

// switch 中的 Pattern matching
String formatted = switch (obj) {
    case Integer i -> "int: " + i;
    case String s -> "string: " + s;
    case null -> "null value";
    default -> "unknown";
};

// Record patterns
record Point(int x, int y) {}
if (obj instanceof Point(int x, int y)) {
    System.out.println(x + ", " + y);
}

// Sequenced Collections
List<String> list = new ArrayList<>();
list.addFirst("first");    // 新方法
list.addLast("last");      // 新方法
list.reversed();           // 反转视图

// String templates（21 中为 preview）
// 可能需要 --enable-preview

// Scoped Values（preview）- 替换 ThreadLocal
ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();
ScopedValue.where(CURRENT_USER, user).run(() -> {
    // CURRENT_USER.get() 在此处可用
});
```

---

## Java 21 → 25 迁移

### 破坏性更改

| Change | Impact |
|--------|--------|
| Security Manager 已删除 | 依赖它的应用程序需要替代安全方法 |
| `sun.misc.Unsafe` 方法已删除 | 使用 `VarHandle` 或 FFM API 替代 |
| 放弃 32 位平台 | 不再支持 x86-32 |
| Record pattern 变量为 final | 不能在 switch 中重新分配模式变量 |
| `ScopedValue.orElse(null)` 不允许 | 必须提供非 null 默认值 |
| 动态 agents 受限 | 需要 `-XX:+EnableDynamicAgentLoading` 标志 |

### 检查 Unsafe 使用

```bash
# 查找 sun.misc.Unsafe 使用
grep -rn "sun\.misc\.Unsafe" --include="*.java" src/

# 查找 Security Manager 使用
grep -rn "SecurityManager\|System\.getSecurityManager" --include="*.java" src/
```

### 采用的新功能

```java
// Scoped Values（Java 25 中为 FINAL）- 替换 ThreadLocal
private static final ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();

public void handleRequest(User user) {
    ScopedValue.where(CURRENT_USER, user).run(() -> {
        processRequest();  // CURRENT_USER.get() 在此处和子线程中可用
    });
}

// Structured Concurrency（Preview，25 中重新设计的 API）
try (StructuredTaskScope.ShutdownOnFailure scope = StructuredTaskScope.open()) {
    Subtask<User> userTask = scope.fork(() -> fetchUser(id));
    Subtask<Orders> ordersTask = scope.fork(() -> fetchOrders(id));

    scope.join();
    scope.throwIfFailed();

    return new Profile(userTask.get(), ordersTask.get());
}

// Stable Values（Preview）- 懒初始化变得简单
private static final StableValue<ExpensiveService> SERVICE =
    StableValue.of(() -> new ExpensiveService());

public void useService() {
    SERVICE.get().doWork();  // 首次访问时初始化，之后缓存
}

// 紧凑对象头 - 自动，无需代码更改
// 对象现在使用 64 位头而不是 128 位（更少内存）

// instanceof 中的基本类型模式（Preview）
if (obj instanceof int i) {
    System.out.println("int value: " + i);
}

// 模块导入声明（Preview）
import module java.sql;  // 从模块导入所有公共类型
```

### 性能改进（自动）

Java 25 包含几个自动性能改进：
- **紧凑对象头**：每个对象 8 字节而不是 16 字节
- **String.hashCode() 常量折叠**：使用 String 键的 Map 查找更快
- **AOT 类加载**：通过 ahead-of-time 缓存更快启动
- **代际 Shenandoah GC**：更好的吞吐量、更低的暂停

### 使用 OpenRewrite 迁移

```bash
# 自动化 Java 25 迁移
mvn -U org.openrewrite.maven:rewrite-maven-plugin:run \
  -Drewrite.recipeArtifactCoordinates=org.openrewrite.recipe:rewrite-migrate-java:LATEST \
  -Drewrite.activeRecipes=org.openrewrite.java.migrate.UpgradeToJava25
```

---

## Spring Boot 迁移

### Spring Boot 2.x → 3.x

**要求：**
- Java 17+（必需）
- Jakarta EE 9+（javax.* → jakarta.*）

**包重命名：**
```java
// 之前（Spring Boot 2.x）
import javax.persistence.*;
import javax.validation.*;
import javax.servlet.*;

// 之后（Spring Boot 3.x）
import jakarta.persistence.*;
import jakarta.validation.*;
import jakarta.servlet.*;
```

**查找并替换：**
```bash
# 查找所有需要迁移的 javax imports
grep -r "import javax\." --include="*.java" src/ | grep -v "javax.crypto" | grep -v "javax.net"
```

**自动化迁移：**
```bash
# 使用 OpenRewrite
mvn -U org.openrewrite.maven:rewrite-maven-plugin:run \
  -Drewrite.recipeArtifactCoordinates=org.openrewrite.recipe:rewrite-spring:LATEST \
  -Drewrite.activeRecipes=org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_0
```

### 依赖更新（Spring Boot 3.x）

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.2</version>
</parent>

<!-- Hibernate 6（自动包含）-->
<!-- Spring Security 6（自动包含）-->
```

### Hibernate 5 → 6 更改

```java
// ID 生成策略已更改
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)  // 首选
private Long id;

// Query 更改
// 之前：createQuery 返回原始类型
// 之后：createQuery 需要类型参数

// 之前
Query query = session.createQuery("from User");

// 之后
TypedQuery<User> query = session.createQuery("from User", User.class);
```

---

## 常见迁移问题

### 问题：反射访问被拒绝

**症状：**
```
java.lang.reflect.InaccessibleObjectException: Unable to make field accessible
```

**修复：**
```bash
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/java.lang.reflect=ALL-UNNAMED
```

### 问题：JAXB ClassNotFoundException

**症状：**
```
java.lang.ClassNotFoundException: javax.xml.bind.JAXBContext
```

**修复：** 添加 JAXB 依赖（见 Java 8→11 部分）

### 问题：Lombok 不工作

**修复：** 更新 Lombok 到最新版本：
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
</dependency>
```

### 问题：Mockito 测试失败

**修复：** 更新 Mockito：
```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.8.0</version>
    <scope>test</scope>
</dependency>
```

---

## 迁移清单

### 迁移前
- [ ] 记录当前 Java 版本
- [ ] 列出所有依赖及其版本
- [ ] 识别内部 API 的使用（`sun.*`、`com.sun.*`）
- [ ] 检查框架兼容性（Spring、Hibernate 等）
- [ ] 备份/创建分支

### 迁移期间
- [ ] 更新构建工具配置
- [ ] 添加缺失的 Jakarta 依赖
- [ ] 修复 `javax.*` → `jakarta.*` imports（如果 Spring Boot 3）
- [ ] 如需要添加 `--add-opens` 标志
- [ ] 更新 Lombok、Mockito 和其他工具
- [ ] 修复编译错误
- [ ] 运行测试

### 迁移后
- [ ] 删除不必要的 `--add-opens` 标志
- [ ] 采用新语言特性（records、var 等）
- [ ] 更新 CI/CD pipeline
- [ ] 记录所做的更改

---

## 快速命令

```bash
# 检查 Java 版本
java -version

# 查找内部 API 使用
grep -rn "sun\.\|com\.sun\." --include="*.java" src/

# 查找 javax imports（用于 Jakarta 迁移）
grep -rn "import javax\." --include="*.java" src/

# 编译并显示前几个错误
mvn clean compile 2>&1 | head -100

# 使用详细的模块警告运行
java --illegal-access=debug -jar app.jar

# OpenRewrite Spring Boot 3 迁移
mvn org.openrewrite.maven:rewrite-maven-plugin:run \
  -Drewrite.recipeArtifactCoordinates=org.openrewrite.recipe:rewrite-spring:LATEST \
  -Drewrite.activeRecipes=org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_0
```

---

## 版本兼容性矩阵

| Framework | Java 8 | Java 11 | Java 17 | Java 21 | Java 25 |
|-----------|--------|---------|---------|---------|---------|
| Spring Boot 2.7.x | ✅ | ✅ | ✅ | ⚠️ | ❌ |
| Spring Boot 3.2.x | ❌ | ❌ | ✅ | ✅ | ✅ |
| Spring Boot 3.4+ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Hibernate 5.6 | ✅ | ✅ | ✅ | ⚠️ | ❌ |
| Hibernate 6.4+ | ❌ | ❌ | ✅ | ✅ | ✅ |
| JUnit 5.10+ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Mockito 5+ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Lombok 1.18.34+ | ✅ | ✅ | ✅ | ✅ | ✅ |

**LTS 支持时间线：**
- Java 21：Oracle 免费支持直到 2028 年 9 月
- Java 25：Oracle 免费支持直到 2033 年 9 月
