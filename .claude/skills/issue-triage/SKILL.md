---
name: issue-triage
description: 对 GitHub issues 进行分类和优先级标记。当用户说 "triage issues"、"check issues"、"review open issues" 或定期维护 GitHub issue backlog 时使用。
---

# Issue Triage 技能

高效地对 Java 项目的 GitHub issues 进行分类、确定优先级。

## 何时使用
- 用户说 "triage issues" / "check recent issues"
- 定期维护工作流
- 假期/休息后（backlog 处理）
- 每周/每月 issue 审查

## 前提条件

**推荐**：配置 GitHub MCP server 以获得最佳 token 使用
```bash
claude mcp add github --transport http \
  https://api.githubcopilot.com/mcp/
```

**替代方案**：使用 `gh` CLI（token 效率较低）

## 工作流

### 1. 获取 Issues

**使用 GitHub MCP**（推荐）：
```
Tool: list_issues
Parameters: {
  "state": "open",
  "sort": "updated",
  "per_page": 10
}
```

**使用 gh CLI**：
```bash
gh issue list --state open --limit 10 --json number,title,labels,body,url
```

### 2. 分类每个 Issue

分析 issue 内容并分配类别：

#### Bug Report ✅
**指标**：
- 有堆栈跟踪或错误消息
- 提供了重现步骤
- 描述了预期 vs 实际行为
- 提到特定版本号

**操作**：
- 标记：`bug`
- 从描述验证可重现性
- 检查重复 bugs
- 如果关键则添加到 milestone

**示例**：
```
Issue #234: "NPE when loading plugin from directory"
- Stack trace: ✅
- Reproduction steps: ✅
- Version info: ✅
→ 标记: bug, high-priority
```

#### Feature Request 💡
**指标**：
- 要求新功能
- 描述了用例
- "It would be nice if..." / "Could you add..."
- 提供了理由

**操作**：
- 标记：`enhancement`
- 评估与项目目标的一致性
- 如果非平凡则标记为讨论
- 请求社区反馈

#### Question/Support ❓
**指标**：
- "How do I..." / "Can someone help..."
- 配置/使用问题
- 不是 bug 或功能请求

**操作**：
- 标记：`question`
- 提供答案或链接到文档
- 建议复杂问题使用 StackOverflow
- 解决后关闭

#### Duplicate 🔄
**搜索类似 issues**：
- 使用 GitHub search：`is:issue <keywords>`
- 检查最近关闭的 issues
- 查找相同的错误消息

**操作**：
- 链接到原始："Duplicate of #123"
- 带有礼貌评论关闭
- 如果报告人有额外信息，请他们在原始 issue 上评论

#### Invalid/Unclear ⚠️
**指标**：
- 缺少关键信息
- 离题或垃圾信息
- 没有足够的上下文继续

**操作**：
- 使用模板请求澄清
- 设置 "needs-more-info" 标记
- 如果 14 天后无响应则自动关闭

### 3. 优先级评估

#### Critical (P0) 🔴
**标准**：
- 安全漏洞
- 数据丢失/损坏风险
- 完全功能故障
- 影响生产系统

**操作**：
- 标记：`critical`
- 立即通知维护者
- 添加到当前 milestone
- 考虑 hotfix 发布

**示例**：
```
- "SQL injection vulnerability in plugin loader"
- "All plugins fail to load after upgrade"
- "ClassLoader leak causes OutOfMemoryError"
```

#### High (P1) 🟠
**标准**：
- 核心功能损坏
- 影响许多用户
- 存在变通方法但很痛苦
- 以前版本的回归

**操作**：
- 标记：`high-priority`
- 添加到下一个 milestone
- 包含在发布说明中

**示例**：
```
- "Plugin dependencies not resolved correctly"
- "Hot reload crashes application"
```

#### Medium (P2) 🟡
**标准**：
- 边缘情况 bug
- 有明确价值的增强
- 文档缺失
- 偶尔影响一些用户

**操作**：
- 标记：`medium-priority`
- 考虑用于未来 milestone
- 适合贡献者

**示例**：
```
- "Improve error message for invalid plugin"
- "Add plugin lifecycle listener"
```

#### Low (P3) 🟢
**标准**：
- 最好有的功能
- 外观问题
- 非常罕见的边缘情况
- 文档改进

**操作**：
- 标记：`low-priority`
- "Contributions welcome" 标记
- Backlog 供将来使用

**示例**：
```
- "Add more examples to README"
- "Typo in JavaDoc"
```

### 4. 响应模板

#### 需要更多信息
```markdown
Thanks for reporting this issue!

To investigate further, could you provide:
- Java version (java -version)
- Library version
- Minimal reproducible example
- Full stack trace (if applicable)
- Configuration files (if relevant)

This will help us diagnose and fix the issue faster.
```

#### 重复
```markdown
Thanks for reporting! This is being tracked in #123.

Closing as duplicate. Feel free to add any additional context
or information to the original issue.
```

#### 不予修复（附理由）
```markdown
Thank you for the suggestion. After consideration, this doesn't
align with the project's current direction because [reason].

Consider [alternative approach] instead, which might better
serve your use case.

If you feel strongly about this, please open a discussion in
our [forum/discussions] to gather community feedback.
```

#### 确认的 Bug
```markdown
Confirmed! This is a valid bug.

I've added it to milestone X.Y and labeled it as [priority].
Contributions welcome if anyone wants to tackle it!

Reproduction verified with:
- Java 17
- Version 3.10.0
- Ubuntu 22.04
```

#### 功能请求 - 考虑中
```markdown
Interesting idea! This aligns with our goal of [project goal].

I've labeled this as 'enhancement' for further discussion.
Community feedback welcome - upvote with 👍 if you'd find
this useful.

Some questions to consider:
- [question 1]
- [question 2]
```

#### 问题已回答
```markdown
To achieve this, you can [solution].

Example:
\`\`\`java
[code example]
\`\`\`

Also check our documentation: [link]

Let me know if this solves your issue!
```

## Token 优化策略

### 批量处理
```bash
# 在一个提示中处理多个 issues
"Triage issues #234-243, categorize and prioritize"
```

**节省**：与逐个处理相比节省约 60% tokens

### 使用结构化的 GitHub MCP 调用
- 一次调用列出 issues → 缓存结果
- 仅在需要时进行细节定向调用
- 批量更新标记

**节省**：与重复 bash 调用相比节省约 40% tokens

### 缓存 Issue 列表
```bash
# 第一个提示
"Fetch the last 20 issues, save list in memory"

# 后续提示
"Analyze issue #5 from cached list"
"Mark #7-#9 as duplicate"
```

### 专注于首条评论 + 最近评论
- 不要阅读整个 50 条评论的线程
- 浏览首条评论获取上下文
- 检查最后 2-3 条评论的更新

## 反模式

❌ **避免**：
```
# 逐个处理
"Check issue #234"
"Now check issue #235"
"Now check issue #236"
→ 在重复上下文加载上浪费 tokens

# 过度分析
阅读整个 100 条评论的线程
检查所有相关的 PR
为每个 issue 深入代码
→ 超过某一点后收益递减

# 过早关闭
未经适当调查关闭 issues
由于搜索不当而遗漏重复
→ 挫败用户，创建重复工作
```

✅ **首选**：
```
# 批量操作
"Triage issues #234-250, categorize, prioritize"

# 快速分类决策
快速分类 → 如需要可以重新访问
大多数 issues 进行表面分析
仅对关键/复杂的进行深入分析

# 彻底的重复搜索
在标记重复之前进行快速关键词搜索
如果存在澄清，链接到特定评论
```

## 自动化机会

### 自动关闭陈旧 issues
```bash
# 90 天无活动且带有 "needs-more-info" 标记的 issues
"Find stale issues (>90 days, needs-more-info label),
suggest closing with polite message"
```

### 按关键词标记
```bash
# 基于内容自动标记
"java.lang.NullPointerException" → bug
"add support for" → enhancement
"how do I" → question
```

### 每周摘要
```bash
# 生成分类摘要
"Summarize issues from last week:
- New bugs: X
- Feature requests: Y
- Questions: Z
- Closed: W"
```

## 与 GitHub 集成

### 使用 GitHub MCP
```javascript
// 结构化工作流
1. list_issues → 获取 open issues
2. get_issue → 每个的详细信息
3. add_labels → 分类
4. create_comment → 响应
5. close_issue → 如果需要
```

### 使用 gh CLI
```bash
# 列出 issues
gh issue list --json number,title,labels,body

# 查看特定 issue
gh issue view 234

# 添加标记
gh issue edit 234 --add-label "bug,high-priority"

# 评论
gh issue comment 234 --body "Thanks for reporting..."

# 关闭
gh issue close 234 --comment "Fixed in v2.1"
```

## 要跟踪的指标

每次分类会话后，报告：

```
📊 Triage Summary
─────────────────
Issues processed: 15
├─ Bugs: 5 (2 critical, 3 high)
├─ Enhancements: 4
├─ Questions: 3
├─ Duplicates: 2
└─ Invalid: 1

Actions taken:
├─ Labeled: 15
├─ Responded: 12
├─ Closed: 3
└─ Milestoned: 5

Time saved: ~45 minutes (vs manual)
Token usage: 3,200 tokens
```

## 最佳实践

1. **定期节奏** - 每周分类防止 backlog
2. **保持尊重** - 用户花时间报告
3. **链接资源** - 文档、相关 issues、示例
4. **询问问题** - 澄清比假设更好
5. **欢迎贡献** - 鼓励社区参与
6. **跟踪模式** - 常见问题表明文档缺失
7. **表扬报告者** - 感谢用户的好 bug 报告
8. **果断关闭** - 不要让 issues 无限期拖延

## 示例工作流

```bash
# 周一早上分类
claude code ~/projects/pf4j

> view .claude/skills/issue-triage/SKILL.md
> "Triage the last 15 issues from pf4j/pf4j,
   categorize, prioritize and suggest responses

[Claude 分析并展示摘要]

> "Apply labels and post the suggested responses"

[Claude 执行操作]

> "Generate summary for release notes"
```

**结果**：15 个 issues 在约 10 分钟内完成分类，手动需要约 45 分钟
