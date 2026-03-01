# Architecture Review Skill

> Java 项目结构、package 和依赖方向的宏观层面分析

## 功能描述

在高层级分析项目架构：
- Package 组织（按层 vs 按功能 vs 六边形架构）
- 层之间的依赖方向
- 模块边界和耦合
- 架构反模式（god packages、贫血领域等）

## 何时使用

- "Review the architecture of this project"
- "Is this package structure good?"
- "Check if we follow clean architecture"
- "Find architectural violations"
- 重大重构工作之前

## 核心概念

### Package 策略

| Strategy | 最适合 | 权衡 |
|----------|----------|-----------|
| By-layer | 小型项目、快速开始 | 分散相关代码 |
| By-feature | 中型项目、清晰的模块 | 需要共享内核 |
| Hexagonal | 复杂领域、可测试性 | 更多仪式 |

### 依赖方向

```
Outer (Framework) → Adapters → Application → Domain (Inner)

Rule: Dependencies point INWARD only
```

## 使用示例

```
You: Review the architecture of this project

Claude: [分析 package 结构]
        [检查依赖方向]
        [识别违规]
        [提供优先级建议]
```

## 检查内容

1. **Package Structure** - 组织、命名一致性
2. **Dependency Direction** - 领域隔离、无框架泄漏
3. **Layer Boundaries** - 适当的关注点分离
4. **Module Boundaries** - 清晰的 API、封装
5. **Scalability** - 功能是否可以被提取？

## 相关技能

- `solid-principles` - 类级别设计（此技能是 package/模块级别）
- `design-patterns` - 实现模式（此技能是结构性的）
- `clean-code` - 代码质量（此技能是架构质量）

## 参考资料

- [Clean Architecture (Uncle Bob)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Package by Feature](https://phauer.com/2020/package-by-feature/)
