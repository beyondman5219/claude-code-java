# Changelog Generator

**Load**: `view .claude/skills/changelog-generator/SKILL.md`

---

## 描述

从遵循既定约定的 git commits 生成 changelogs。自动检测版本控制风格和 changelog 格式（来自现有项目文件 CLAUDE.md、git tags、CHANGELOG.md）。

---

## 使用场景

- "Generate changelog since last release"
- "What changed since v3.14.0?"
- "Update CHANGELOG.md for version 3.16"
- "Preview unreleased changes"

---

## 示例

```
> view .claude/skills/changelog-generator/SKILL.md
> "Generate changelog for pf4j"
→ Detects pf4j format and SemVer style, outputs matching changelog
```

---

## 核心功能

- **版本检测**：SemVer（x.y.z）、双组件（x.y）、CalVer（YYYY.MM）
- **格式检测**：适应现有 CHANGELOG.md 风格
- **引用式链接**：清晰的 `[#123]` 格式，底部带有定义
- **版本比较链接**：自动生成 GitHub compare URLs
- **Legacy 支持**：适用于没有明确约定的项目

---

## 检测优先级

1. CLAUDE.md `## Versioning` 部分
2. Git tags 模式分析
3. 现有 CHANGELOG.md 格式
4. 询问用户（最后手段）

---

## 注意事项 / 提示

- 与 conventional commits 配合效果最好（与 git-commit skill 配对）
- 对于 legacy 项目，建议将版本约定添加到 CLAUDE.md
- 更新时保留现有的链接定义
