## 派发子 agent 需要多 agent 支持

将以下内容添加到 Codex 配置（`~/.codex/config.toml`）：

```toml
[features]
multi_agent = true
```

这会启用 `dispatching-parallel-agents` 和 `subagent-driven-development` 等技能使用的多 agent 工具。你能获得哪些工具取决于模型预设选择的多 agent 版本（当前预设运行 V2；旧预设运行 V1）。如果它们与任何表格不一致，请相信实际的工具列表——包括这张表。

- **派生：**使用 `spawn_agent {fork_turns: "none"}` 为子 agent 提供干净的上下文；默认值 `"all"` 会将完整的对话记录复制给子 agent。在 Codex 0.145+ 中，`~/.codex/agents/` 下的角色文件会通过 `agent_type` 附加到隔离 fork。完整历史 fork 接受 `model` 和 `reasoning_effort` 覆盖（只会拒绝 `agent_type`）——隔离 fork 是 SDD 的默认设置，原因是上下文卫生，而不是因为覆盖项需要它们。
- **修复轮次：**使用 `followup_task` 恢复实现者——它会传递你的消息、触发一轮操作，并透明地重新加载被 harness 驱逐的子 agent。不要因为认为派生出的 agent 无法再次接收消息，就重新派发一个实现者；在 V2 中，它始终可以再次接收消息。
- **生命周期：**V2 没有 `close_agent`。当需要槽位时，已完成的子 agent 会自动被驱逐；不关闭它们不会产生任何成本。只有 V1 会话有 `close_agent`——在那里，当审查者返回后关闭它们，并在每个实现者通过审查后关闭它。
- **模型名称：**绝不要在没有对照你当前的派生允许清单（spawn allowlist）核验的情况下，把来自技能、表格或旧会话的模型名称复制进 `spawn_agent`——V2 只接受具备 V2 能力的预设，对其余预设会直接报错。

## 等待子 agent

`wait_agent` 是事件订阅，而不是轮询：长时间等待会在子 agent 产生邮箱活动的瞬间唤醒，并且延迟与短等待相同。短超时轮询没有收益，只会增加一次工具调用和一次上下文计费；在测量的会话中，大约三分之二的 wait 调用都是超时的短轮询。

- 当你还有本地工作时，完全不要等待。已完成子 agent 的最终答案会被推送到你的邮箱，并在你的下一轮中到达。
- 当你确实空闲且仍有子 agent 未完成时，以有界的时间段等待：使用 `wait_agent`，`timeout_ms` 为 300000-600000（5-10 分钟）。每个时间段结束后——无论是被唤醒还是超时——发布一行状态，运行 `list_agents`，并追踪任何已经完成但没有报告的子 agent。绝不要叠加短于 5 分钟的轮询；事件订阅在有界时间段内同样会迅速唤醒。
- 完成消息无法唤醒空闲的控制器（消息会被投递，但不会触发新一轮）；覆盖这个空闲窗口是 `wait_agent` 唯一的作用。没有活动而超时的时间段，是你进行对账的信号，而不是缩短下一次等待。

## 派生时的模型路由

你发出的每一个 `spawn_agent`——包括你自己作为扇出的一部分运行时——都要根据模型选择规则明确设置 `model` 和 `reasoning_effort`。只设置 `model` 是个陷阱：子 agent 的 effort 会静默重置为该模型的默认值，而不是你的设置。

请你的伙伴在 `~/.codex/config.toml` 中添加机器级后备配置，让任何漏网的派生操作都路由到有意选择的层级，而不是静默继承当前会话最昂贵的模型：

```toml
[agents]
default_subagent_model = "<a mid-tier model from your spawn allowlist>"
default_subagent_reasoning_effort = "medium"
```

## 环境检测

需要创建工作树或完成分支的技能，应在继续之前使用只读 git 命令检测环境：

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → 已经处于链接工作树中（跳过创建）
- `BRANCH` 为空 → 处于 detached HEAD 状态（无法在沙箱中创建分支/推送/创建 PR）

参见 `using-git-worktrees` 第 0 步，以及 `finishing-a-development-branch` 第 1 步，了解每个技能如何使用这些信号。

## Codex App 收尾

当沙箱阻止分支/推送操作（外部管理的工作树处于 detached HEAD 状态）时，agent 会提交所有工作，并告知用户使用 App 的原生控件：

- **“创建分支”**——命名分支，然后通过 App UI 提交/推送/创建 PR
- **“移交到本地”**——将工作转移到用户的本地检出

agent 仍然可以运行测试、暂存文件，并输出建议的分支名称、提交消息和 PR 描述，供用户复制。
