---
name: architecture-review
description: 在宏观层面分析 Java 项目架构 - package 结构、模块边界、依赖方向和分层。当用户询问 "review architecture"、"check structure"、"package organization" 或评估代码库是否遵循 clean architecture 原则时使用。
---

# Architecture Review 技能

在宏观层面分析项目结构 - packages、modules、layers 和 boundaries。

## 何时使用
- 用户询问 "review the architecture" / "check project structure"
- 评估 package 组织
- 检查层之间的依赖方向
- 识别架构违规
- 评估 clean/hexagonal architecture 合规性

---

## 快速参考：架构异味

| 异味 | 症状 | 影响 |
|-------|---------|--------|
| 按层分组的 package 膨胀 | `service/` 中有 50+ 个类 | 难以找到相关代码 |
| Domain → Infra 依赖 | Entity 导入 `@Repository` | 核心逻辑绑定到框架 |
| 循环依赖 | A → B → C → A | 不可测试、脆弱 |
| 上帝 package | `util/` 或 `common/` 增长 | 错放代码的堆场 |
| 泄漏的抽象 | Controller 知道 SQL | 层边界被违反 |

---

## Package 组织策略

### Package-by-Layer（传统）

```
com.example.app/
├── controller/
│   ├── UserController.java
│   ├── OrderController.java
│   └── ProductController.java
├── service/
│   ├── UserService.java
│   ├── OrderService.java
│   └── ProductService.java
├── repository/
│   ├── UserRepository.java
│   ├── OrderRepository.java
│   └── ProductRepository.java
└── model/
    ├── User.java
    ├── Order.java
    └── Product.java
```

**优点**：熟悉、小型项目简单
**缺点**：分散相关代码、不可扩展、难以提取模块

### Package-by-Feature（推荐）

```
com.example.app/
├── user/
│   ├── UserController.java
│   ├── UserService.java
│   ├── UserRepository.java
│   └── User.java
├── order/
│   ├── OrderController.java
│   ├── OrderService.java
│   ├── OrderRepository.java
│   └── Order.java
└── product/
    ├── ProductController.java
    ├── ProductService.java
    ├── ProductRepository.java
    └── Product.java
```

**优点**：相关代码在一起、易于提取、清晰的边界
**缺点**：横切关注点可能需要共享内核

### Hexagonal/Clean Architecture

```
com.example.app/
├── domain/                    # 纯业务逻辑（无框架导入）
│   ├── model/
│   │   └── User.java
│   ├── port/
│   │   ├── in/               # 用例（被驱动）
│   │   │   └── CreateUserUseCase.java
│   │   └── out/              # Repositories（驱动）
│   │       └── UserRepository.java
│   └── service/
│       └── UserDomainService.java
├── application/               # 用例实现
│   └── CreateUserService.java
├── adapter/
│   ├── in/
│   │   └── web/
│   │       └── UserController.java
│   └── out/
│       └── persistence/
│           ├── UserJpaRepository.java
│           └── UserEntity.java
└── config/
    └── BeanConfiguration.java
```

**关键规则**：依赖指向内部（adapters → application → domain）

---

## 依赖方向规则

### 黄金法则

```
┌─────────────────────────────────────────┐
│              Frameworks                 │  ← Outer（易变）
├─────────────────────────────────────────┤
│           Adapters (Web, DB)            │
├─────────────────────────────────────────┤
│         Application Services            │
├─────────────────────────────────────────┤
│          Domain (Core Logic)            │  ← Inner（稳定）
└─────────────────────────────────────────┘

依赖必须仅指向内部。
内层绝不能了解外层。
```

### 需要标记的违规

```java
// ❌ Domain 依赖 infrastructure
package com.example.domain.model;

import org.springframework.data.jpa.repository.JpaRepository;  // 框架泄漏！
import javax.persistence.Entity;  // Domain 中的 JPA！

@Entity
public class User {
    // Domain 被持久化关注点污染
}

// ❌ Domain 依赖 adapter
package com.example.domain.service;

import com.example.adapter.out.persistence.UserJpaRepository;  // 错误的方向！

// ✅ Domain 定义 port，adapter 实现
package com.example.domain.port.out;

public interface UserRepository {  // 纯接口，无 JPA
    User findById(UserId id);
    void save(User user);
}
```

---

## 架构审查清单

### 1. Package 结构
- [ ] 清晰的组织策略（按层、按功能或六边形）
- [ ] 模块间命名一致
- [ ] 没有 `util/` 或 `common/` packages 无界增长
- [ ] 功能 packages 是内聚的（相关代码在一起）

### 2. 依赖方向
- [ ] Domain 零框架导入（Spring、JPA、Jackson）
- [ ] Adapters 依赖 domain，反之亦然
- [ ] packages 之间没有循环依赖
- [ ] 清晰的依赖层次结构

### 3. 层边界
- [ ] Controllers 不包含业务逻辑
- [ ] Services 不知道 HTTP（没有 HttpServletRequest）
- [ ] Repositories 不泄漏到 controllers
- [ ] 边界使用 DTO，内部使用 domain 对象

### 4. 模块边界
- [ ] 每个模块有清晰的公共 API
- [ ] 内部类是 package-private
- [ ] 跨模块通信通过接口
- [ ] 不"跨模块"访问内部

### 5. 可扩展性指标
- [ ] 可以提取功能到独立服务吗？（微服务就绪）
- [ ] 边界是强制的还是仅是约定？
- [ ] 添加功能是否需要触及多个 packages？

---

## 常见反模式

### 1. 大泥球

```
src/main/java/com/example/
└── app/
    ├── User.java
    ├── UserController.java
    ├── UserService.java
    ├── UserRepository.java
    ├── Order.java
    ├── OrderController.java
    ├── ...（一个 package 中 100+ 个文件）
```

**修复**：引入 package 结构（从按功能开始）

### 2. Util 堆场

```
util/
├── StringUtils.java
├── DateUtils.java
├── ValidationUtils.java
├── SecurityUtils.java
├── EmailUtils.java      # 应该在 notification 模块
├── OrderCalculator.java # 应该在 order domain
└── UserHelper.java      # 应该在 user domain
```

**修复**：将领域逻辑移动到适当的模块，仅保留真正的通用 utils

### 3. 贫血领域模型

```java
// 领域对象只是数据
public class Order {
    private Long id;
    private List<OrderLine> lines;
    private BigDecimal total;
    // 只有 getters/setters，没有行为
}

// 所有逻辑在 "service" 中
public class OrderService {
    public void addLine(Order order, Product product, int qty) { ... }
    public void calculateTotal(Order order) { ... }
    public void applyDiscount(Order order, Discount discount) { ... }
}
```

**修复**：将行为移动到领域对象（富领域模型）

### 4. Domain 中的框架耦合

```java
package com.example.domain;

@Entity  // JPA
@Data    // Lombok
@JsonIgnoreProperties(ignoreUnknown = true)  // Jackson
public class User {
    @Id @GeneratedValue
    private Long id;

    @NotBlank  // Validation
    private String email;
}
```

**修复**：分离领域模型与持久化/API 模型

---

## 分析命令

审查架构时，检查：

```bash
# Package 结构概览
find src/main/java -type d | head -30

# 最大的 packages（潜在的上帝 packages）
find src/main/java -name "*.java" | xargs dirname | sort | uniq -c | sort -rn | head -10

# 检查 domain 中的框架导入
grep -r "import org.springframework" src/main/java/*/domain/ 2>/dev/null
grep -r "import javax.persistence" src/main/java/*/domain/ 2>/dev/null

# 查找循环依赖（查找双向导入）
# 检查 package A 是否从 B 导入，B 是否从 A 导入
```

---

## 建议格式

报告发现时：

```markdown
## Architecture Review: [Project Name]

### Structure Assessment
- **Organization**: Package-by-layer / Package-by-feature / Hexagonal
- **Clarity**: Clear / Mixed / Unclear

### Findings

| Severity | Issue | Location | Recommendation |
|----------|-------|----------|----------------|
| High | Domain 导入 Spring | `domain/model/User.java` | 提取纯领域模型 |
| Medium | 上帝 package | `util/`（23 个类）| 分发到功能模块 |
| Low | 命名不一致 | `service/` vs `services/` | 标准化为 `service/` |

### Dependency Analysis
[描述依赖流、发现的违规]

### Recommendations
1. [最高优先级修复]
2. [次要优先级]
3. [最好有]
```

---

## Token 优化

对于大型代码库：
1. 从 `find` 开始了解结构
2. 仅检查 domain package 的框架导入
3. 采样 2-3 个功能进行模式分析
4. 不要读取每个文件 - 查找模式
