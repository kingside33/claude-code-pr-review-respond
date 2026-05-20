# claude-code-pr-review-respond

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](SKILL.md)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-orange.svg)](https://claude.ai/code)

> [English Version](README.md)

一个 [Claude Code](https://claude.ai/code) 技能，用于自动化处理 GitHub Pull Request 上的代码审查评论。它会获取审查评论、分类、实施修复、运行测试、提交、推送、用中文回复并解决所有讨论串。

## 功能

1. **获取** 所有 PR 审查评论（通过 GitHub REST + GraphQL API）
2. **分类** 每条评论为：已修复 / 需要修复 / 不采纳
3. **实施** 代码修复，运行测试，提交，推送
4. **回复** 每条审查评论（使用中文）
5. **解决** 回复后立即解决每个讨论串

## 为什么是技能而不是普通提示词

- 系统化分类审查评论，而非临时处理
- 正确使用 GitHub GraphQL API 进行评论串级别的回复和解决
- 一致的中文回复格式
- 规范的提交信息格式，包含 Co-Authored-By 署名
- 强制执行"先测试再提交"原则
- 自动检测不同技术栈的测试运行器

## 安装

### 方式一：Git 克隆（推荐）

```bash
git clone https://github.com/WangZixu/claude-code-pr-review-respond \
  ~/.claude/skills/pr-review-respond
```

然后在 `~/.claude/CLAUDE.md` 中引用：

```markdown
- **pr-review-respond** (`~/.claude/skills/pr-review-respond/SKILL.md`) — PR 审查回复自动化。触发: `/pr-review-respond`
```

### 方式二：npx skills add

```bash
npx skills add https://github.com/WangZixu/claude-code-pr-review-respond
```

### 方式三：手动下载

将 `SKILL.md` 下载到任意位置，添加到 `CLAUDE.md`：

```markdown
- PR 审查回复 → read `~/skills/pr-review-respond/SKILL.md`
```

## 使用方法

```
/pr-review-respond <owner/repo> <PR编号>
```

如果不提供参数，技能会从 `git remote get-url origin` 和当前分支自动检测。

## 环境要求

- [GitHub CLI](https://cli.github.com/) (`gh`) 已安装并认证 (`gh auth login`)
- Git 仓库需关联 GitHub 远程仓库
- 需要仓库的写入权限
- 项目需包含测试套件

## 工作流程

### 第一步 — 获取审查评论

使用 `gh api` 获取所有 PR 审查评论（REST 接口）和讨论串解决状态（GraphQL 接口）。

### 第二步 — 分类

| 分类 | 判断标准 | 操作 |
|------|---------|------|
| 已修复 | 修复已存在于之前的提交中 | 回复说明哪个提交，然后解决 |
| 需要修复 | 有效问题，需要代码更改 | 修复 → 测试 → 提交 → 推送 → 回复 → 解决 |
| 不采纳 | 无效、超出范围或设计决策 | 回复说明理由，然后解决 |

### 第三步 — 实施修复

进行最小化代码更改，自动检测并运行测试套件，按照 conventional commit 格式提交（含 `Co-Authored-By` 署名），然后推送。

### 第四步 — 回复并解决

使用 GraphQL 在每个审查评论串下用中文回复，然后立即解决该讨论串。逐条处理，不批量操作。

### 第五步 — 更新文档

如项目维护了 `docs/change-log.md`、`docs/progress.md` 等文档，根据 CLAUDE.md 约定选择性更新。

## 回复格式

所有回复使用中文：

- **已修复**：已修复：\<描述\>。Commit: \<短SHA\>
- **不采纳**：不采纳：\<理由\>
- **需要修复（修复后）**：已修复：\<描述\>。Commit: \<短SHA\>

## 仓库结构

```
.
├── SKILL.md                # 核心技能定义
├── README.md               # 英文文档
├── README.zh-CN.md         # 中文文档
├── LICENSE                 # MIT 许可证
├── CHANGELOG.md            # 版本历史
├── CONTRIBUTING.md         # 贡献指南
├── examples/               # 项目级修复示例
│   ├── fetch-race-condition.md
│   ├── time-range-validation.md
│   ├── thread-safe-trimming.md
│   └── file-writer-cleanup.md
├── references/             # 补充参考
│   ├── github-api.md
│   └── commit-conventions.md
└── .github/                # Issue 和 PR 模板
```

## 贡献

详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 许可证

MIT — 详见 [LICENSE](LICENSE)。
