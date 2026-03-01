# 测试策略

> 如何测试和验证 claude-code-java 脚本

## 当前方法：简单测试脚本

对于 MVP 阶段，我们使用一个简单的 bash 测试脚本来验证所有设置脚本正常工作。

### 运行测试

```bash
./scripts/test-all.sh
```

### 测试内容

| 脚本 | 验证 |
|--------|-------------|
| `link-skills.sh` | 创建 `.claude/`，符号链接指向工作区 |
| `generate-claude-md.sh` | 创建包含内容的 `CLAUDE.md` |
| `configure-mcp.sh` | 模板文件存在 |

### 测试理念

- 测试在临时目录中运行（自动清理）
- 零外部依赖
- 快速执行（< 2 秒）
- 清晰的通过/失败输出

## 未来选项

### 选项 1：bats-core（推荐用于增长）

[bats-core](https://github.com/bats-core/bats-core) - Bash 自动化测试系统

**何时采用：**
- 10+ 个测试用例
- 多个贡献者
- 想要更好的测试组织（describe/it 块）

**示例：**
```bash
@test "link-skills creates symlink" {
    run ./scripts/link-skills.sh "$TEST_DIR"
    [ "$status" -eq 0 ]
    [ -L "$TEST_DIR/.claude/skills" ]
}
```

**安装：** `npm install -g bats` 或 `brew install bats-core`

### 选项 2：GitHub Actions CI

**何时采用：**
- 项目在 GitHub 上公开
- 想要在 PR 上自动验证
- 多个贡献者

**示例工作流（`.github/workflows/test.yml`）：**
```yaml
name: Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: chmod +x scripts/*.sh
      - run: ./scripts/test-all.sh
```

### 选项 3：Pre-commit Hook

**何时采用：**
- 想要在 commit 之前发现问题
- 仅本地验证

**设置：**
```bash
# .git/hooks/pre-commit
#!/bin/bash
./scripts/test-all.sh || exit 1
```

## 决策框架

| 阶段 | 推荐方法 |
|-------|---------------------|
| MVP（现在） | 简单测试脚本 |
| v0.3+ 带贡献者 | 添加 bats-core |
| 公开发布 | 添加 GitHub Actions |
| 团队采用 | 添加 pre-commit hooks |

## 添加新测试

添加新脚本时，在 `test-all.sh` 中添加相应的测试：

```bash
# 测试 N：new-script.sh
echo "Testing new-script.sh..."
"$SCRIPT_DIR/new-script.sh" "$TEST_DIR" > /dev/null 2>&1
check "[ -f '$TEST_DIR/expected-output' ]" "expected output created"
echo ""
```

## 手动测试清单

对于难以自动化的更改：

- [ ] 在真实的 Java 项目上运行 `setup-project.sh`
- [ ] 验证技能符号链接在 Claude Code 中工作
- [ ] 在新目录上测试（没有现有的 `.claude/`）
- [ ] 在带有现有 `.claude/skills` 的目录上测试
