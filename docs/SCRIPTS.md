# 脚本指南

> 设置脚本的工作原理以及如何扩展它们

## 可用脚本

| 脚本 | 用途 |
|--------|---------|
| `setup-project.sh` | 编排完整项目设置（运行下面的所有脚本） |
| `link-skills.sh` | 创建 `.claude/` 目录并符号链接技能 |
| `generate-claude-md.sh` | 从模板生成 `CLAUDE.md` |
| `configure-mcp.sh` | 生成 MCP 配置并可选地添加服务器 |
| `configure-settings.sh` | 复制带有预批准命令的 Claude Code 设置 |
| `test-all.sh` | 运行所有测试以验证脚本工作 |

## 使用方法

### 完整设置（推荐）

```bash
cd /path/to/claude-code-java
./scripts/setup-project.sh /path/to/your-java-project
```

### 单独脚本

```bash
# 仅链接技能
./scripts/link-skills.sh /path/to/your-java-project

# 仅生成 CLAUDE.md
./scripts/generate-claude-md.sh /path/to/your-java-project

# 仅配置 MCP
./scripts/configure-mcp.sh /path/to/your-java-project

# 仅配置 settings
./scripts/configure-settings.sh /path/to/your-java-project
```

### 运行测试

```bash
./scripts/test-all.sh
```

## 约定

所有脚本遵循相同的结构以确保一致性和可靠性。

### 路径解析模式

每个脚本都以以下内容开头：

```bash
#!/bin/bash
set -e

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
WORKSPACE_DIR="$(dirname "$SCRIPT_DIR")"

PROJECT_DIR="$(cd "${1:-.}" && pwd)"
```

**为什么这很重要：**
- `SCRIPT_DIR` - scripts/ 目录的绝对路径
- `WORKSPACE_DIR` - claude-code-java 根目录的绝对路径
- `PROJECT_DIR` - 目标项目的绝对路径（参数或当前目录）

这确保脚本无论以下情况都能正确工作：
- 你从哪里运行它们
- 路径是否包含空格
- 路径中的符号链接

### 禁止 `cd` 规则

脚本不应使用 `cd` 来更改目录。相反，使用绝对路径：

```bash
# 好的做法
[ -d "$PROJECT_DIR/.claude" ] && mkdir -p "$PROJECT_DIR/.claude"

# 坏的做法
cd "$PROJECT_DIR"
[ -d .claude ] && mkdir -p .claude
```

**原因：** `cd` 后，到模板/工作区资源的相对路径会中断。

### 模板文件

模板位于 `templates/` 中并使用 `{{PLACEHOLDER}}` 语法：

```
templates/
├── CLAUDE.md.template        # {{PROJECT_NAME}}, {{REPO_NAME}}, {{DATE}}
├── mcp-config.json.template  # {{PROJECT_ROOT}}, {{GITHUB_REPO}}
├── MCP_CONFIG.md.template    # {{PROJECT_ROOT}}, {{GITHUB_REPO}}
└── settings.json.template    # 预批准的 Maven/Git 命令
```

脚本使用 `sed` 来替换占位符：

```bash
sed -e "s/{{PROJECT_NAME}}/$PROJECT_NAME/g" \
    -e "s/{{DATE}}/$DATE/g" \
    "$TEMPLATE_FILE" > "$OUTPUT_FILE"
```

### 错误处理

所有脚本使用 `set -e` 在第一个错误时退出。对于不应停止执行的检查：

```bash
# 如果文件丢失，这将退出脚本
[ ! -f "$FILE" ] && echo "Error" && exit 1

# 即使命令失败也会继续
some_command || true
```

### 输出消息

使用一致的格式：

```bash
echo "✅ 成功消息"
echo "❌ 错误消息"
echo "ℹ️  信息消息"
echo "⚠️  警告消息"
```

## 添加新脚本

1. 在 `scripts/` 中创建文件：

```bash
#!/bin/bash
# new-script.sh - 简要描述
# 用法：./new-script.sh [project-directory]

set -e

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
WORKSPACE_DIR="$(dirname "$SCRIPT_DIR")"

PROJECT_DIR="$(cd "${1:-.}" && pwd)"

# 使用绝对路径的逻辑在这里
```

2. 使其可执行：

```bash
chmod +x scripts/new-script.sh
```

3. 在 `test-all.sh` 中添加测试：

```bash
# 测试 N：new-script.sh
echo "Testing new-script.sh..."
"$SCRIPT_DIR/new-script.sh" "$TEST_DIR" > /dev/null 2>&1
check "expected result" [ -f "$TEST_DIR/expected-file" ]
echo ""
```

4. 更新此文档。

## 脚本依赖关系

```
setup-project.sh
    ├── link-skills.sh      （无依赖）
    ├── generate-claude-md.sh
    │       └── templates/CLAUDE.md.template
    ├── configure-mcp.sh
    │       ├── templates/mcp-config.json.template
    │       └── templates/MCP_CONFIG.md.template
    └── configure-settings.sh
            └── templates/settings.json.template
```

## 故障排除

### "Template not found"（模板未找到）

脚本无法找到模板文件。检查：
- 你从工作区目录运行，或
- 脚本正确解析 WORKSPACE_DIR

### "Permission denied"（权限被拒绝）

脚本需要执行权限：

```bash
chmod +x scripts/*.sh
```

### Windows 上的符号链接问题

Windows 需要开发者模式或管理员权限才能创建符号链接。考虑使用 WSL 或复制文件而不是符号链接。
