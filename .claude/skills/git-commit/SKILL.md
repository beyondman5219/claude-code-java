---
name: git-commit
description: 为 Java 项目生成 conventional commit 消息。当用户说 "commit"、"create commit"、"commit changes" 或在完成需要提交的代码更改后使用。
---

# Git Commit 消息技能

为 Java 项目生成符合规范、信息丰富的 commit 消息。

## 何时使用
- 在进行代码更改后
- 用户说 "commit this" / "commit changes" / "create commit"
- 在创建 PR 之前

## 格式标准

使用 Conventional Commits 格式：
```
<type>(<scope>): <subject>

<body>

<footer>
```

### 类型（Java 上下文）
- **feat**：新功能（新 API、新功能）
- **fix**：Bug 修复
- **refactor**：代码重构（无功能更改）
- **test**：添加/更新测试
- **docs**：仅文档
- **perf**：性能改进
- **build**：Maven/Gradle 更改
- **chore**：维护（依赖更新等）

### 范围示例（Java 特定）
- 模块名称：`core`、`api`、`plugin-loader`
- 组件：`PluginManager`、`ExtensionFactory`
- 区域：`lifecycle`、`dependencies`、`security`

### 主题规则
- 祈使语气："Add support" 而不是 "Added support"
- 结尾无句号
- 最多 50 个字符
- 类型后小写

### 正文（可选但推荐）
- 解释 WHAT 和 WHY，而不是 HOW
- 72 个字符换行
- 引用 issues："Fixes #123" / "Relates to #456"

## 示例

### 简单修复
```
fix(plugin-loader): 防止插件目录缺失时出现 NPE

在访问插件目录之前检查 null，以避免初始化期间的
NullPointerException。

Fixes #234
```

### 带有破坏性更改的功能
```
feat(api): 添加对插件依赖版本控制的支持

BREAKING CHANGE: PluginDescriptor 现在需要语义化版本控制
格式 (x.y.z)，而不是自由格式的版本字符串。

Closes #567
```

### 重构
```
refactor(core): 提取插件验证逻辑

将验证逻辑从 PluginManager 移动到单独的
PluginValidator 类，以提高可测试性和关注点分离。
```

### 添加测试
```
test(plugin-loader): 添加插件加载的集成测试

添加全面的集成测试，涵盖：
- 从目录加载
- 从 JAR 加载
- 无效插件的错误处理
```

### 构建/依赖更新
```
build(deps): 升级 Spring Boot 到 3.2.1

将 Spring Boot 从 3.1.0 更新到 3.2.1 以获取安全补丁
和性能改进。
```

## 工作流

1. **分析更改** 使用 `git diff --staged`
2. **识别范围** 从修改的文件
3. **确定类型** 基于更改性质
4. **生成消息** 遵循格式
5. **执行 commit**：`git commit -m "message"`

## Token 优化

- 读取暂存的更改一次：`git diff --staged --stat` + 针对性文件 diff
- 除非必要，否则不要读取整个文件
- 使用简洁的正文 - 最多 2-3 行
- 将多个小更改批量为逻辑 commits

## 反模式

❌ 避免：
- "fix stuff" / "update code" / "changes"
- "WIP" commits（除非明确要求）
- 混合不相关的更改（使用单独的 commits）
- 消息中过度详细的技术实现

✅ 好的 commits：
- 单个逻辑更改
- 清晰、可搜索的主题
- 适用时引用 issues
- 解释业务价值

## 与 GitHub 集成

commit 后，建议下一步：
- "推送更改？"
- "为 issue #X 创建 PR？"
- "继续下一个任务？"

## Java 项目的常见模式

### 添加新功能
```
feat(extension): 添加对优先级扩展的支持

允许扩展指定执行的优先级顺序。
优先级较高的扩展先运行。

Closes #123
```

### 修复 Bug
```
fix(classloader): 解决嵌套 JAR 中的资源查找

ClassLoader.getResource() 对于从插件 JAR 加载的 JAR 中的
资源（嵌套 JAR）失败。通过实现适当的资源解析链修复。

Fixes #456
```

### 依赖更新
```
build(deps): 将 slf4j 从 1.7.30 升级到 2.0.9

将 SLF4J 更新到最新的稳定版本。不需要 API 更改，
因为我们只使用稳定的 API。
```

### 文档改进
```
docs(readme): 添加插件开发快速入门指南

添加创建第一个插件的分步指南：
- 项目设置
- 实现 Plugin 接口
- 构建和测试
```

### 性能优化
```
perf(plugin-loader): 缓存插件描述符

缓存解析的插件描述符以避免重复 I/O
和解析。将插件加载时间减少约 40%。

Related to #789
```

## 多文件更改

当更改跨越多个组件时：

```
refactor(core): 重新组织插件生命周期管理

- 将生命周期状态机提取到单独的类
- 将验证逻辑移动到 validators 包
- 更新测试以反映新结构

此重构提高了可测试性和关注点分离，
而不更改外部 API。

Related to #111, #222
```

## 破坏性更改

始终使用 BREAKING CHANGE footer：

```
feat(api)!: 用 Plugin.initialize() 替换 Plugin.start()

BREAKING CHANGE: Plugin.start() 方法已重命名
为 Plugin.initialize() 以获得更好的语义清晰度。所有
插件实现必须更新其代码。

迁移指南：在所有 Plugin 实现中，将 @Override start() 替换为 @Override
initialize()。

Closes #999
```

## 快速参考卡

| 更改类型 | 类型 | 示例范围 |
|-------------|------|---------------|
| 新功能 | feat | api、core、loader |
| Bug 修复 | fix | plugin-loader、lifecycle |
| 重构 | refactor | core、utils |
| 测试 | test | integration、unit |
| 文档 | docs | readme、javadoc |
| 构建 | build | maven、deps |
| 性能 | perf | classloader、cache |
| 维护 | chore | ci、tooling |

## 参考资料

- [Conventional Commits 规范](https://www.conventionalcommits.org/)
