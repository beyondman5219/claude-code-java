---
name: maven-dependency-audit
description: 审计 Maven 依赖的过时版本、安全漏洞和冲突。当用户说 "check dependencies"、"audit dependencies"、"outdated deps" 或在发布之前使用。
---

# Maven Dependency Audit 技能

审计 Maven 依赖的更新、漏洞和冲突。

## 何时使用
- 用户说 "check dependencies" / "audit dependencies" / "outdated dependencies"
- 发布之前
- 定期维护（建议每月）
- 安全公告之后

## 审计工作流

1. **检查更新** - 查找过时的依赖
2. **分析树** - 查找冲突和重复
3. **安全扫描** - 检查漏洞
4. **报告** - 带有优先操作摘要

---

## 1. 检查过时的依赖

### 命令
```bash
mvn versions:display-dependency-updates
```

### 输出分析
```
[INFO] The following dependencies in Dependencies have newer versions:
[INFO]   org.slf4j:slf4j-api ......................... 1.7.36 -> 2.0.9
[INFO]   com.fasterxml.jackson.core:jackson-databind . 2.14.0 -> 2.16.1
[INFO]   org.junit.jupiter:junit-jupiter ............. 5.9.0 -> 5.10.1
```

### 分类更新

| Category | 标准 | 操作 |
|----------|----------|--------|
| **Security** | 新版本中有 CVE 修复 | 立即更新 |
| **Major** | x.0.0 变化 | 审查 changelog，彻底测试 |
| **Minor** | x.y.0 变化 | 通常安全，测试 |
| **Patch** | x.y.z 变化 | 安全，最少测试 |

### 也检查插件更新
```bash
mvn versions:display-plugin-updates
```

---

## 2. 分析依赖树

### 完整树
```bash
mvn dependency:tree
```

### 过滤特定依赖
```bash
mvn dependency:tree -Dincludes=org.slf4j
```

### 查找冲突
查找：
```
[INFO] +- com.example:module-a:jar:1.0:compile
[INFO] |  \- org.slf4j:slf4j-api:jar:1.7.36:compile
[INFO] +- com.example:module-b:jar:1.0:compile
[INFO] |  \- org.slf4j:slf4j-api:jar:2.0.9:compile (omitted for conflict)
```

**标志：**
- `(omitted for conflict)` - Maven 解决的版本冲突
- `(omitted for duplicate)` - 相同版本，无问题
- 同一库的多个版本 - 潜在的运行时问题

### 分析未使用的依赖
```bash
mvn dependency:analyze
```

输出：
```
[WARNING] Used undeclared dependencies found:
[WARNING]    org.slf4j:slf4j-api:jar:2.0.9:compile
[WARNING] Unused declared dependencies found:
[WARNING]    commons-io:commons-io:jar:2.11.0:compile
```

---

## 3. 安全漏洞扫描

### 选项 A：OWASP Dependency-Check（推荐）

添加到 pom.xml：
```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>9.0.7</version>
</plugin>
```

运行：
```bash
mvn dependency-check:check
```

输出：`target/dependency-check-report.html` 中的 HTML 报告

### 选项 B：Maven Dependency Plugin
```bash
mvn dependency:analyze-report
```

### 选项 C：GitHub Dependabot
如果使用 GitHub，在仓库设置中启用 Dependabot 警报。

### 严重性级别

| CVSS 分数 | 严重性 | 操作 |
|------------|----------|--------|
| 9.0 - 10.0 | Critical | 立即更新 |
| 7.0 - 8.9 | High | 数天内更新 |
| 4.0 - 6.9 | Medium | 数周内更新 |
| 0.1 - 3.9 | Low | 方便时更新 |

---

## 4. 生成审计报告

### 输出格式

```markdown
## Dependency Audit Report

**Project:** {project-name}
**Date:** {date}
**Total Dependencies:** {count}

### Security Issues

| Dependency | Current | CVE | Severity | Fixed In |
|------------|---------|-----|----------|----------|
| log4j-core | 2.14.0 | CVE-2021-44228 | Critical | 2.17.1 |

### Outdated Dependencies

#### Major Updates (Review Required)
| Dependency | Current | Latest | Notes |
|------------|---------|--------|-------|
| slf4j-api | 1.7.36 | 2.0.9 | API 变化，见迁移指南 |

#### Minor/Patch Updates (Safe)
| Dependency | Current | Latest |
|------------|---------|--------|
| junit-jupiter | 5.9.0 | 5.10.1 |
| jackson-databind | 2.14.0 | 2.16.1 |

### Conflicts Detected
- slf4j-api: 1.7.36 vs 2.0.9 (resolved to 2.0.9)

### Unused Dependencies
- commons-io:commons-io:2.11.0 (考虑删除)

### Recommendations
1. **Immediate:** 更新 log4j-core 修复 CVE-2021-44228
2. **This sprint:** 更新 minor/patch 版本
3. **Plan:** 评估 slf4j 2.x 迁移
```

---

## 常见场景

### 场景：发布前检查
```bash
# 快速检查
mvn versions:display-dependency-updates -q

# 完整审计
mvn versions:display-dependency-updates && \
mvn dependency:analyze && \
mvn dependency-check:check
```

### 场景：查找为何包含依赖
```bash
mvn dependency:tree -Dincludes=commons-logging
```

### 场景：强制特定版本（解决冲突）
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>2.0.9</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 场景：排除传递依赖
```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>some-library</artifactId>
    <version>1.0</version>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

---

## Token 优化

- 使用 `-q`（quiet）标志减少冗余输出
- 查找特定 deps 时使用 `-Dincludes=groupId:artifactId` 过滤
- 分别运行命令并总结发现
- 不要粘贴整个依赖树 - 总结冲突

---

## 快速命令参考

| Task | Command |
|------|---------|
| 过时的 deps | `mvn versions:display-dependency-updates` |
| 过时的插件 | `mvn versions:display-plugin-updates` |
| 依赖树 | `mvn dependency:tree` |
| 查找特定 dep | `mvn dependency:tree -Dincludes=groupId` |
| 未使用的 deps | `mvn dependency:analyze` |
| 安全扫描 | `mvn dependency-check:check` |
| 更新版本 | `mvn versions:use-latest-releases` |
| 更新 snapshots | `mvn versions:use-latest-snapshots` |

---

## 更新策略

### 保守（推荐用于生产）
1. 自由更新 patch 版本
2. 基本测试后更新 minor 版本
3. Major 版本需要迁移计划

### 激进（用于活跃开发）
```bash
# 更新所有到最新（谨慎使用！）
mvn versions:use-latest-releases
mvn versions:commit  # 或 versions:revert
```

### 选择性
```bash
# 更新特定依赖
mvn versions:use-latest-versions -Dincludes=org.junit.jupiter
```
