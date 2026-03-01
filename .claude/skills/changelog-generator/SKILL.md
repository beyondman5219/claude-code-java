---
name: changelog-generator
description: 从 git commits 生成 changelogs。当用户说 "generate changelog"、"update changelog"、"what changed since last release" 或准备新版本之前使用。
---

# Changelog Generator 技能

从 conventional commits 为 Java 项目生成 changelogs。

## 何时使用
- 发布之前
- 用户说 "generate changelog" / "update changelog" / "what changed since last release"
- 完成里程碑之后

## 版本约定检测

按以下优先顺序检测版本控制风格：

### 1. 检查 CLAUDE.md（如果存在）
```bash
grep -A5 "## Versioning" CLAUDE.md 2>/dev/null
```

查找明确的约定：
```markdown
## Versioning
This project uses Semantic Versioning (x.y.z).
Tag format: `release-x.y.z`
```

### 2. 回退：从 git tags 检测
```bash
git tag --sort=-version:refname | head -10
```

| 检测到的模式 | Versioning style |
|------------------|------------------|
| `v3.15.0`, `3.15.0` | SemVer (x.y.z) |
| `release-3.15.0` | SemVer with prefix |
| `v2.1`, `2.1` | Two-component (x.y) |
| `2026.01`, `26.1` | CalVer |
| No pattern | 询问用户 |

### 3. 回退：从 CHANGELOG.md 检测
```bash
grep -E "^\#+ \[.*\]" CHANGELOG.md | head -5
```

从现有条目提取版本格式。

### 4. 最后手段：询问用户
```
No versioning convention detected. Which format does this project use?
- Semantic Versioning (x.y.z) - e.g., 3.15.0
- Two-component (x.y) - e.g., 2.1
- Calendar Versioning - e.g., 2026.01
```

### 支持的版本控制风格

| Style | Format | Tag 示例 | Version bump |
|-------|--------|--------------|--------------|
| SemVer | x.y.z | `v3.15.0`, `release-3.15.0` | major.minor.patch |
| Two-component | x.y | `v2.1`, `2.1` | major.minor |
| CalVer | YYYY.MM[.patch] | `2026.01`, `2026.01.1` | year.month[.patch] |

### Legacy 项目（没有 versioning 部分的 CLAUDE.md）

如果 CLAUDE.md 存在但没有版本信息：
1. 不要假设 - 从 tags/changelog 检测
2. 如果检测到，可选择建议添加到 CLAUDE.md：
   ```
   Detected versioning: SemVer (x.y.z) with tag prefix 'release-'
   Want me to add this to CLAUDE.md for future reference?
   ```

## 输出格式

支持两种格式 - 从现有 CHANGELOG.md 检测或询问用户偏好。

### Format A: Keep a Changelog（h2 版本）
```markdown
# Changelog

## [Unreleased]

## [1.2.0] - 2026-01-29

### Added
- [#123]: New feature for plugin dependencies

### Changed
- [#456]: Improved performance of plugin loading

### Fixed
- [#234]: Resolved NPE when directory missing
```

### Format B: pf4j style（h3 版本）
```markdown
## Change Log

### [Unreleased][unreleased]

### [3.15.0] - 2026-01-29

#### Added
- [#123]: New feature for plugin dependencies

#### Changed
- [#456]: Improved performance of plugin loading

#### Fixed
- [#234]: Resolved NPE when directory missing
```

## 引用式链接（推荐）

使用引用式链接以获得更清晰、更易读的条目：

```markdown
#### Fixed
- [#648]: Restore missing `module-info.class` in multi-release JAR
- [#625]: Fix exception handling inconsistency in `startPlugin()`

#### Added
- [#629]: Validate dependency state on plugin start
- [#633]: Allow customization of `PluginClassLoader` parent delegation

<!-- At the bottom of the file -->
[#648]: https://github.com/user/repo/issues/648
[#633]: https://github.com/user/repo/pull/633
[#629]: https://github.com/user/repo/pull/629
[#625]: https://github.com/user/repo/pull/625
```

**好处：**
- 更清晰易读（内联没有长 URL）
- 链接定义一次，可重用
- 更易于编写和维护

## 版本比较链接

在底部添加比较链接以便轻松查看 diff：

```markdown
[unreleased]: https://github.com/user/repo/compare/release-3.15.0...HEAD
[3.15.0]: https://github.com/user/repo/compare/release-3.14.1...release-3.15.0
[3.14.1]: https://github.com/user/repo/compare/release-3.14.0...release-3.14.1
```

**模式：** `[version]: https://github.com/{owner}/{repo}/compare/{previous-tag}...{current-tag}`

## 部分顺序

适应现有文件，或使用此默认顺序：

| Section | 何时使用 |
|---------|-------------|
| Fixed | Bug 修复 |
| Changed | 对现有功能的更改 |
| Added | 新功能 |
| Deprecated | 即将删除的功能 |
| Removed | 已删除的功能 |
| Security | 漏洞修复 (CVEs) |

注意：pf4j 使用 Fixed → Changed → Added → Removed。Keep a Changelog 使用 Added → Changed → Deprecated → Removed → Fixed → Security。

**规则：如果存在现有文件，遵循其顺序。**

## 映射 Conventional Commits 到 Changelog

| Commit Type | Changelog Section |
|-------------|-------------------|
| feat | Added |
| fix | Fixed |
| perf | Changed |
| refactor | Changed |
| build(deps) | Changed 或 Security（如果是 CVE） |
| BREAKING CHANGE | Changed（带粗体注释） |
| deprecate | Deprecated |

## 工作流

1. **检查现有 CHANGELOG.md**
   ```bash
   cat CHANGELOG.md | head -20
   ```
   检测格式（h2 vs h3 版本、部分顺序、链接样式）。

2. **确定版本范围**
   ```bash
   # 查找最后一个 tag
   git describe --tags --abbrev=0

   # 列出最近的 tags
   git tag --sort=-version:refname | head -5
   ```

3. **获取自上次发布以来的 commits**
   ```bash
   git log v3.14.1..HEAD --oneline
   ```

4. **提取 issue/PR 引用**
   查找模式：`#123`、`fixes #123`、`closes #123`、`(#123)`

5. **生成 changelog 条目**
   - 按部分分组
   - 使用引用式链接
   - 添加版本比较链接

6. **建议版本升级**（基于检测到的版本控制风格）

   **SemVer (x.y.z):**
   - BREAKING CHANGE → Major (3.0.0 → 4.0.0)
   - feat → Minor (3.14.0 → 3.15.0)
   - 仅 fix → Patch (3.14.0 → 3.14.1)

   **Two-component (x.y):**
   - BREAKING CHANGE → Major (2.0 → 3.0)
   - feat/fix → Minor (2.1 → 2.2)

   **CalVer (YYYY.MM):**
   - 新月份 → 2026.01 → 2026.02
   - 同月，新发布 → 2026.01 → 2026.01.1

## Token 优化

- 使用 `git log --oneline` 进行初始扫描
- 仅在怀疑 BREAKING CHANGE 时获取完整 body
- 重用文件中现有的链接定义
- 不要重新读取整个 changelog - 只需在前面添加新部分

## 示例：完整工作流

**输入：** 用户说 "generate changelog for next release"

**Step 1:** 检查现有格式
```bash
head -30 CHANGELOG.md
```
→ 检测到 pf4j style（h3 版本，Fixed 在前）

**Step 2:** 查找版本范围
```bash
git describe --tags --abbrev=0
```
→ `release-3.15.0`

**Step 3:** 获取 commits
```bash
git log release-3.15.0..HEAD --oneline
```
→ 发现 5 个 commits

**Step 4:** 生成条目
```markdown
### [Unreleased][unreleased]

#### Fixed
- [#650]: Fix memory leak in extension factory

#### Changed
- [#651]: Rename `LegacyExtension*` to `IndexedExtension*`

#### Added
- [#652]: Add support for plugin priority ordering
```

**Step 5:** 生成链接定义
```markdown
[#652]: https://github.com/pf4j/pf4j/pull/652
[#651]: https://github.com/pf4j/pf4j/pull/651
[#650]: https://github.com/pf4j/pf4j/issues/650
```

**Step 6:** 更新版本比较链接
```markdown
[unreleased]: https://github.com/pf4j/pf4j/compare/release-3.15.0...HEAD
```

**Step 7:** 建议版本
```
Suggested: 3.16.0 (minor - has new feature)
```

## 处理边缘情况

### 没有 conventional commits
在 "Changed" 下列出原始消息：
```markdown
#### Changed
- Updated plugin loading mechanism
- Refactored test utilities
```

### 安全修复
```markdown
#### Security
- [#618], [#623]: Fix path traversal vulnerabilities in ZIP extraction
```

### 破坏性更改
```markdown
#### Changed
- **BREAKING**: [#645] Renamed `LegacyExtension*` classes to `IndexedExtension*`
```

### 同一修复的多个 issues
```markdown
- [#630], [#631]: Set `failedException` when plugin validation fails
```

## 与现有 CHANGELOG.md 集成

1. **读取现有文件**以检测：
   - 标题级别（## 或 ### 用于版本）
   - 部分顺序
   - 链接样式（引用或内联）
   - 现有链接定义

2. **在 `[Unreleased]` 部分后插入新版本**

3. **合并链接定义** - 添加新的，保留现有的

4. **更新 `[unreleased]` 比较链接**指向新版本

## 快速参考

| 用户说 | 操作 |
|-----------|--------|
| "generate changelog" | 自最后一个 tag 的完整 changelog |
| "changelog since v3.14" | 从特定版本开始 |
| "what's unreleased" | 预览未发布的更改 |
| "update changelog for 3.16" | 为版本生成并插入 |
| "add changelog entry for #123" | 单个 issue 条目 |
