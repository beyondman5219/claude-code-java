# Maven Dependency Audit

**Load**: `view .claude/skills/maven-dependency-audit/SKILL.md`

---

## 描述

审计 Maven 依赖的过时版本、安全漏洞和冲突。使用标准 Maven 插件 - 不需要额外的工具。

---

## 使用场景

- "Check for outdated dependencies"
- "Audit dependencies before release"
- "Find security vulnerabilities in pom.xml"
- "Why is commons-logging in my project?"

---

## 示例

```
> view .claude/skills/maven-dependency-audit/SKILL.md
> "Audit dependencies for pf4j"
→ Runs checks, categorizes updates by severity, generates report
```

---

## 使用的工具

| Tool | Purpose |
|------|---------|
| `mvn versions:display-dependency-updates` | 查找过时的依赖 |
| `mvn dependency:tree` | 分析依赖树 |
| `mvn dependency:analyze` | 查找未使用的依赖 |
| `mvn dependency-check:check` | 安全漏洞扫描（OWASP） |

---

## 注意事项 / 提示

- 每月或每次发布前运行
- Patch 更新通常是安全的；主要更新需要审查
- 使用 `-Dincludes=groupId` 过滤大型依赖树
- 考虑启用 GitHub Dependabot 以进行自动警报
