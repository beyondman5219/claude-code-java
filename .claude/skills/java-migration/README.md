# Java Migration

**Load**: `view .claude/skills/java-migration/SKILL.md`

---

## 描述

在主要 LTS 版本之间升级 Java 项目的分步指南（8→11→17→21→25）。包括破坏性更改、移除的 API、采用的新功能以及特定于框架的迁移（Spring Boot、Hibernate）。

---

## 使用场景

- "Upgrade project to Java 25"
- "Migrate from Java 21 to 25"
- "Spring Boot 3 migration"
- "What breaks when upgrading to Java 25?"
- "Fix javax.xml.bind not found"

---

## 示例

```
> view .claude/skills/java-migration/SKILL.md
> "Upgrade this project from Java 11 to 21"
→ Analyzes code, identifies breaking changes, provides step-by-step fixes
```

---

## 涵盖的迁移路径

| From | To | 关键更改 |
|------|-----|-------------|
| Java 8 | Java 11 | JAXB 移除、模块系统、内部 API |
| Java 11 | Java 17 | Records、sealed classes、强封装 |
| Java 17 | Java 21 | Virtual threads、pattern matching、sequenced collections |
| Java 21 | Java 25 | Security Manager 移除、Unsafe 移除、Scoped Values final |
| Spring Boot 2.x | 3.x | javax.* → jakarta.*、需要 Java 17 |
| Hibernate 5 | 6 | Query API 更改、ID 生成 |

---

## 使用的工具

| Tool | Purpose |
|------|---------|
| `grep` | 查找已弃用的 API 使用 |
| `mvn compile` | 识别编译错误 |
| OpenRewrite | 自动 Spring Boot 3 迁移 |
| `--add-opens` | 修复反射访问问题 |

---

## 注意事项 / 提示

- 始终迁移 LTS → LTS（8→11→17→21→25）
- 首先更新 Lombok、Mockito 到最新版本
- 使用 OpenRewrite 进行自动迁移
- 每步后彻底测试
- Java 25 LTS 支持到 2033 年 9 月

## 参考资料

- [Oracle JDK 25 Migration Guide](https://docs.oracle.com/en/java/javase/25/migrate/)
- [Oracle JDK 25 Release Notes](https://www.oracle.com/java/technologies/javase/25-relnote-issues.html)
- [OpenRewrite Java Migration Recipes](https://docs.openrewrite.org/recipes/java/migrate)
- [Spring Boot 3.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)
