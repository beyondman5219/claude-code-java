# 技能

技能是可复用的提示，教 Claude Java 开发的特定模式。

## 结构约定

每个技能文件夹包含：

| 文件 | 目的 | 受众 |
|------|---------|----------|
| `SKILL.md` | Claude 的指令 | AI（使用 `view` 加载） |
| `README.md` | 文档、示例、提示 | 人类（入职） |

## 可用技能

### 工作流
| 技能 | 描述 |
|-------|-------------|
| [git-commit](git-commit/) | 用于 Java 项目的 Conventional commit 消息 |
| [changelog-generator](changelog-generator/) | 从 git commits 生成 changelogs |
| [issue-triage](issue-triage/) | GitHub issue 分类和分类 |

### 代码质量
| 技能 | 描述 |
|-------|-------------|
| [java-code-review](java-code-review/) | 系统的 Java 代码审查清单 |
| [api-contract-review](api-contract-review/) | REST API 审计：HTTP 语义、版本控制、兼容性 |
| [concurrency-review](concurrency-review/) | 线程安全、竞态条件、@Async、Virtual Threads |
| [performance-smell-detection](performance-smell-detection/) | 代码级性能异味（streams、boxing、regex） |
| [test-quality](test-quality/) | JUnit 5 + AssertJ 测试模式 |
| [maven-dependency-audit](maven-dependency-audit/) | 审计依赖的更新和漏洞 |
| [security-audit](security-audit/) | OWASP Top 10、输入验证、注入预防 |

### 架构与设计
| 技能 | 描述 |
|-------|-------------|
| [architecture-review](architecture-review/) | 宏观级审查：包、模块、层、边界 |
| [solid-principles](solid-principles/) | 带有 Java 示例的 S.O.L.I.D. 原则 |
| [design-patterns](design-patterns/) | Factory、Builder、Strategy、Observer、Decorator 等 |
| [clean-code](clean-code/) | DRY、KISS、YAGNI、命名、重构 |

### 框架与数据
| 技能 | 描述 |
|-------|-------------|
| [spring-boot-patterns](spring-boot-patterns/) | Spring Boot 最佳实践 |
| [java-migration](java-migration/) | Java 版本升级指南（8→11→17→21） |
| [jpa-patterns](jpa-patterns/) | JPA/Hibernate 模式（N+1、lazy loading、事务） |
| [logging-patterns](logging-patterns/) | 结构化日志（JSON）、SLF4J、MDC、AI 友好格式 |

## 添加新技能

### 开始之前

根据现有技能验证你的技能想法：

- [ ] **无显著重叠** - 检查上表是否有类似技能
- [ ] **清晰的级别** - 微观（函数）/ 中观（类）/ 宏观（包）/ 框架 / 横切
- [ ] **清晰的类型** - 审计（审查现有代码）或 模板（展示如何编写）
- [ ] **独特价值** - 它添加了什么不存在的？
- [ ] **专注的范围** - 可以在一个会话中应用（<15 个清单项）

> 📖 **完整指南：** [docs/SKILL_GUIDELINES.md](../../docs/SKILL_GUIDELINES.md)

### 实现步骤

1. 创建文件夹：`.claude/skills/<skill-name>/`
2. 创建 `SKILL.md`，包含 Claude 的指令
3. 创建 `README.md`，包含人类文档（使用现有 README 作为模板）
4. 更新此表
5. 更新主 README.md

## 使用方法

技能由 Claude Code 根据上下文自动加载。你也可以直接调用它们：

```bash
# 自动 - Claude 检测何时使用技能
> "Commit these changes"        # 加载 git-commit
> "Review this code for SOLID"  # 加载 solid-principles

# 手动 - 使用斜杠命令调用
> /git-commit
> /solid-principles
```

## 了解更多

- [Claude Code 技能文档](https://code.claude.com/docs/en/skills) - 关于创建和使用技能的官方指南
