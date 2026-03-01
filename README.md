# claude-code-java

> 为 Java 项目打造的可复用 AI 开发基础设施，针对 Claude Code 进行优化

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

*本项目与 Anthropic 无关联。*

## 这是什么？

[Anthropic 的智能编码工具](https://docs.anthropic.com/en/docs/claude-code)的可复用组件集合。该项目的核心是一套 **skills**（技能）（结构化的 markdown 文件，为 Claude 提供领域知识和工作流），但也包括项目模板、MCP 服务器配置和设置脚本。

**适合谁使用？** 使用 Claude Code 的 Java 开发者，希望在代码审查、测试、提交和架构决策等常见任务中获得一致、高质量的 AI 辅助。

## 目的

以 AI 驱动的开发工作流程，重点关注：
- **Token 效率** - 旨在减少迭代次数并降低 token 使用量
- **可复用模式** - 跨项目的可复用工作流
- **Java 生态系统** - 专为 Java/Maven 开发定制
- **渐进式采用** - 从小规模开始，按需扩展

## 快速开始

### 1. 克隆此工作区
```bash
git clone https://github.com/decebals/claude-code-java.git ~/projects/claude-code-java
cd ~/projects/claude-code-java
chmod +x scripts/*.sh
```

### 2. 设置你的 Java 项目
```bash
./scripts/setup-project.sh ~/projects/your-java-project
```

这会创建包含符号链接技能的 `.claude/` 目录，生成 `CLAUDE.md`，并配置 settings。

**更喜欢手动设置？** 只需复制或符号链接你想要的技能：
```bash
mkdir -p your-project/.claude/skills

# 复制特定技能
cp -r ~/projects/claude-code-java/.claude/skills/java-code-review your-project/.claude/skills/

# 或符号链接所有技能
ln -s ~/projects/claude-code-java/.claude/skills/* your-project/.claude/skills/
```

### 3. 与 Claude Code 一起使用
```bash
cd ~/projects/your-java-project
claude

# 技能基于上下文自动加载，或直接调用：
> /git-commit
> /java-code-review
```

## 可用技能 (18 个)

技能由 Claude Code 根据上下文自动加载。

### 工作流
| 技能 | 触发示例 |
|-------|------------------|
| [**git-commit**](.claude/skills/git-commit/) | "提交这些更改", "创建 commit" |
| [**changelog-generator**](.claude/skills/changelog-generator/) | "生成 changelog", "自发布以来有什么变化" |
| [**issue-triage**](.claude/skills/issue-triage/) | "分类 issues", "检查开放 issues" |

### 代码质量
| 技能 | 触发示例 |
|-------|------------------|
| [**java-code-review**](.claude/skills/java-code-review/) | "审查这段代码", "检查这个 PR" |
| [**api-contract-review**](.claude/skills/api-contract-review/) | "审查 API", "检查 REST endpoints" |
| [**concurrency-review**](.claude/skills/concurrency-review/) | "检查线程安全", "审查异步代码" |
| [**performance-smell-detection**](.claude/skills/performance-smell-detection/) | "检查性能", "查找慢代码" |
| [**test-quality**](.claude/skills/test-quality/) | "添加测试", "提高覆盖率" |
| [**maven-dependency-audit**](.claude/skills/maven-dependency-audit/) | "检查依赖", "审计 deps" |
| [**security-audit**](.claude/skills/security-audit/) | "安全审查", "检查 OWASP", "漏洞" |

### 架构与设计
| 技能 | 触发示例 |
|-------|------------------|
| [**architecture-review**](.claude/skills/architecture-review/) | "审查架构", "检查 package 结构" |
| [**solid-principles**](.claude/skills/solid-principles/) | "检查 SOLID", "单一职责" |
| [**design-patterns**](.claude/skills/design-patterns/) | "使用 factory pattern", "实现 strategy" |
| [**clean-code**](.claude/skills/clean-code/) | "清理这段代码", "重构" |

### 框架与数据
| 技能 | 触发示例 |
|-------|------------------|
| [**spring-boot-patterns**](.claude/skills/spring-boot-patterns/) | "创建 controller", "Spring Boot 帮助" |
| [**java-migration**](.claude/skills/java-migration/) | "升级到 Java 21", "从 Java 8 迁移" |
| [**jpa-patterns**](.claude/skills/jpa-patterns/) | "N+1 问题", "LazyInitializationException" |
| [**logging-patterns**](.claude/skills/logging-patterns/) | "添加日志", "调试这个流程", "分析日志" |

查看 [.claude/skills/README.md](.claude/skills/README.md) 获取完整文档，查看 [docs/SCRIPTS.md](docs/SCRIPTS.md) 了解设置脚本选项。

## 项目结构

```
claude-code-java/
├── README.md                    # 本文件
├── LICENSE                      # MIT 许可证
├── .gitignore                   # Git 忽略规则
├── .claude/
│   └── skills/                  # 18 个可复用技能（见上文可用技能）
├── docs/                        # 指南和最佳实践
│   ├── DESIGN_PRINCIPLES.md     # 核心理念
│   ├── RED_FLAGS.md             # 需警惕的警告信号
│   ├── SAFE_WORKFLOWS.md        # 分步安全工作流
│   ├── SCRIPTS.md               # 脚本文档
│   ├── SKILL_GUIDELINES.md      # 如何创建新技能
│   └── TESTING.md               # 测试策略
├── templates/
│   ├── CLAUDE.md.template       # 项目模板
│   ├── mcp-config.json.template # MCP 配置模板
│   ├── MCP_CONFIG.md.template   # MCP 文档模板
│   └── settings.json.template   # Claude Code 设置（预批准的命令）
└── scripts/
    ├── setup-project.sh         # 完整项目设置（编排器）
    ├── link-skills.sh           # 将技能符号链接到项目
    ├── generate-claude-md.sh    # 生成 CLAUDE.md
    ├── configure-mcp.sh         # 配置 MCP 服务器
    └── test-all.sh              # 运行所有测试
```

## 典型工作流

1. 将技能链接到你的 Java 项目
2. 在项目目录中启动 Claude Code
3. 加载与当前任务相关的技能
4. 使用自然语言执行工作流
5. 测量结果（使用的 token、节省的时间）

## 成功指标

追踪这些指标以验证有效性：

- **Token 减少**：追踪与手动工作流相比的改进
- **时间节省**：测量每个任务的改进前后
- **可复用性**：使用技能的项目数量
- **质量**：代码审查反馈、测试覆盖率

## 系统要求

- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Java 11+ 项目（推荐 Java 17+）
- Git 版本控制
- Maven 或 Gradle 构建工具
- （可选）[GitHub MCP server](https://github.com/github/github-mcp-server) 用于 issue 管理

## 包含内容

- 18 个技能（工作流、代码质量、架构、框架）
- 设置自动化脚本
- 项目模板
- YAML frontmatter 用于自动技能检测

## 用于自动化代码审查

这些技能不仅用于代码生成 — 它们还用作自动化代码审查的**单一真实来源**，通过 [`skill-review`](https://github.com/decebals/skill-review)，一个可复用的 GitHub Actions 工作流，根据 Claude Code 在开发期间使用的相同技能来评估 pull requests。

相同的技能。从生成到审查。参见 [`skill-review-sandbox`](https://github.com/decebals/skill-review-sandbox) 获取工作示例。

## 贡献

技能基于实际使用不断演进。试用它们，提出问题，分享有效的方法。

1. 在你的项目中试用这些技能
2. 为建议提出 issues
3. 分享 token 节省/改进

## 文档

查看 [docs/](docs/) 获取详细指南：
- [DESIGN_PRINCIPLES.md](docs/DESIGN_PRINCIPLES.md) - 核心理念
- [SAFE_WORKFLOWS.md](docs/SAFE_WORKFLOWS.md) - 推荐工作流
- [RED_FLAGS.md](docs/RED_FLAGS.md) - 需警惕的警告信号
- [SKILL_GUIDELINES.md](docs/SKILL_GUIDELINES.md) - 如何创建新技能

## 许可证

MIT License - 可自由使用，按需修改。
