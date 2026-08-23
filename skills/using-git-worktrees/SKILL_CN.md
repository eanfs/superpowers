---
name: using-git-worktrees
description: 在开始需要与当前工作区隔离的功能开发或执行实现计划之前使用——通过原生工具或 git worktree 回退方案确保存在隔离的工作区
---

# 使用 Git Worktree

## 概述

确保工作在隔离的工作区中进行。优先使用平台的原生 worktree 工具。只有在没有可用原生工具时，才回退到手动 git worktree。

**核心原则：** 先检测现有隔离状态，再使用原生工具，最后回退到 git。绝不要与运行环境对抗。

**开始时宣布：**“我正在使用 using-git-worktrees 技能来设置隔离的工作区。”

## 步骤 0：检测现有隔离

**在创建任何内容之前，检查你是否已经处于隔离的工作区中。**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**子模块防护：** 子模块内部同样满足 `GIT_DIR != GIT_COMMON`。在得出“已经处于 worktree 中”的结论前，确认你不在子模块内：

```bash
# If this returns a path, you're in a submodule, not a worktree — treat as normal repo
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**如果 `GIT_DIR != GIT_COMMON`（且不是子模块）：** 你已经处于链接的 worktree 中。跳到步骤 2（项目设置）。不要再创建另一个 worktree。

根据分支状态报告：
- 在某个分支上：“Already in isolated workspace at `<path>` on branch `<name>`。”
- HEAD 游离：“Already in isolated workspace at `<path>` (detached HEAD, externally managed). Branch creation needed at finish time。”

**如果 `GIT_DIR == GIT_COMMON`（或处于子模块内）：** 你位于普通的仓库检出中。

你的指令中是否已经表明了 worktree 偏好？如果没有，在创建 worktree 之前请求同意：

> “你希望我设置一个隔离的 worktree 吗？它可以保护你当前的分支不受更改影响。”

尊重已有的明确偏好，不要再次询问。如果用户拒绝同意，就在当前目录中工作并跳到步骤 2。

## 步骤 1：创建隔离的工作区

**你有两种机制。按以下顺序尝试。**

### 1a. 原生 Worktree 工具（首选）

用户已经请求隔离的工作区（步骤 0 中已同意）。你是否已经有创建 worktree 的方式？它可能是名为 `EnterWorktree`、`WorktreeCreate` 的工具、`/worktree` 命令或 `--worktree` 标志。如果有，使用它并跳到步骤 2。

原生工具会自动处理目录放置、分支创建和清理。在有原生工具时使用 `git worktree add` 会创建运行环境无法查看或管理的幽灵状态。

只有在没有可用原生 worktree 工具时，才继续执行步骤 1b。

### 1b. Git Worktree 回退方案

**仅当步骤 1a 不适用时使用**——也就是没有可用的原生 worktree 工具。使用 git 手动创建 worktree。

#### 目录选择

按以下优先级顺序执行。用户明确的偏好始终优先于观察到的文件系统状态。

1. **检查你的指令中是否声明了 worktree 目录偏好。** 如果用户已经指定，直接使用，不要询问。

2. **检查现有的项目本地 worktree 目录：**
   ```bash
   ls -d .worktrees 2>/dev/null     # Preferred (hidden)
   ls -d worktrees 2>/dev/null      # Alternative
   ```
   如果找到，使用它。如果两者都存在，优先 `.worktrees`。

3. **如果没有其他可用指引，** 默认在项目根目录使用 `.worktrees/`。

#### 安全验证（仅项目本地目录）

**创建 worktree 之前，必须验证目录已被忽略：**

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果未被忽略：** 将其加入 .gitignore，提交更改，然后继续。

**为什么关键：** 防止意外将 worktree 内容提交到仓库。

#### 创建 Worktree

```bash
# Determine path based on chosen location
path="$LOCATION/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**沙箱回退：** 如果 `git worktree add` 因权限错误而失败（沙箱拒绝），告诉用户沙箱阻止了 worktree 创建，改为在当前目录中工作。然后在当前目录执行设置和基线测试。

## 步骤 2：项目设置

自动检测并运行适当的设置：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## 步骤 3：验证干净的基线

运行测试，确保工作区从干净状态开始：

```bash
# Use project-appropriate command
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：** 报告失败情况，询问是否继续或进行调查。

**如果测试通过：** 报告已准备就绪。

### 报告

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## 快速参考

| 情况 | 操作 |
|-----------|--------|
| 已经处于链接的 worktree 中 | 跳过创建（步骤 0） |
| 位于子模块中 | 视为普通仓库（步骤 0 防护） |
| 有原生 worktree 工具可用 | 使用它（步骤 1a） |
| 没有原生工具 | 回退到 Git worktree（步骤 1b） |
| `.worktrees/` 存在 | 使用它（验证已被忽略） |
| `worktrees/` 存在 | 使用它（验证已被忽略） |
| 两者都存在 | 使用 `.worktrees/` |
| 两者都不存在 | 检查指令文件，然后默认使用 `.worktrees/` |
| 目录未被忽略 | 加入 .gitignore + 提交 |
| 创建时发生权限错误 | 沙箱回退，在当前目录中工作 |
| 基线测试失败 | 报告失败 + 询问 |
| 没有 package.json/Cargo.toml | 跳过依赖安装 |

## 常见合理化借口

| 借口 | 现实 |
|--------|---------|
| "I'm obviously not in a worktree — no need to check" | 执行步骤 0。运行环境创建的隔离和子模块都可能欺骗肉眼；检测命令会给出确定结果。 |
| "`git worktree add` is quicker than hunting for a native tool" | 原生工具（例如 `EnterWorktree`）负责放置、分支和清理。绕过它是第一大错误——它会创建运行环境无法查看或管理的幽灵状态。 |
| "The worktree directory is surely ignored already" | 运行 `git check-ignore`。未被忽略的 worktree 目录会把整个目录树提交进仓库。 |
| "Any directory name works" | 明确指令优先于现有的项目本地目录，后者优先于 `.worktrees/` 默认值。 |
| "The workspace is fresh — baseline tests can wait" | 脏的基线会让之后的每个失败都变得含糊不清。现在运行测试；是否在失败后继续由你的用户伙伴决定。 |
