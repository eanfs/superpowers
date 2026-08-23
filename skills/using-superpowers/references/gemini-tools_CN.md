# Gemini CLI 工具映射

技能以动作（“调度子代理”“创建待办事项”“读取文件”）进行描述。在 Gemini CLI 上，这些动作对应以下工具。

| 技能请求的动作 | Gemini CLI 等价工具 |
|----------------------|----------------------|
| 读取文件 | `read_file` |
| 一次读取多个文件 | `read_many_files` |
| 创建新文件 | `write_file` |
| 编辑文件 | `replace` |
| 运行 Shell 命令 | `run_shell_command` |
| 搜索文件内容 | `grep_search` |
| 按名称查找文件 | `glob` |
| 列出文件和子目录 | `list_directory` |
| 获取 URL | `web_fetch` |
| 搜索网页 | `google_web_search` |
| 调用技能 | `activate_skill` |
| 调度子代理（`Subagent (general-purpose):` 模板） | 使用 `agent_name: "generalist"` 的 `invoke_agent`（可通过 `@generalist` 聊天语法调用——见[子代理支持](#subagent-support)） |
| 并行调度多个任务 | 在同一响应中发起多个 `invoke_agent` 调用 |
| 任务跟踪（“创建待办事项”“标记完成”） | `write_todos`（状态：pending、in_progress、completed、cancelled、blocked） |

## 指令文件

当技能提到“你的指令文件”时，在 Gemini CLI 上指的是 **`GEMINI.md`**。Gemini CLI 会分层加载 `GEMINI.md`：全局文件位于 `~/.gemini/GEMINI.md`，项目级文件位于工作区目录及其祖先目录中，访问这些目录下的文件时还会加载子目录中的 `GEMINI.md` 文件。

## 个人技能目录

用户级技能位于 **`~/.gemini/skills/`**，其中 **`~/.agents/skills/`** 是跨运行时别名（与 Codex 和 Copilot CLI 共享）。当同一作用域同时存在两个目录时，`.agents/skills/` 优先。每个技能都是一个包含 `SKILL.md` 的子目录，其中带有 `name` 和 `description` frontmatter。

## 子代理支持

Gemini CLI 通过 `invoke_agent` 工具调度子代理，该工具接收 `agent_name` 和 `prompt` 参数。相同的调度也通过聊天语法快捷方式提供：输入 `@generalist <prompt>` 等价于调用 `invoke_agent`，并将 `agent_name` 设为 `generalist`。内置代理名称包括 `generalist`、`cli_help`、`codebase_investigator`，以及（启用浏览器工具时的）`browser_agent`。

技能使用 `Subagent (general-purpose):` 调度，并且可以引用提示词模板文件（例如 `superpowers:subagent-driven-development` 的 `./implementer-prompt.md`），也可以提供内联提示词。在 Gemini CLI 上：

| 技能调度形式 | Gemini CLI 等价方式 |
|----------------------|----------------------|
| 引用 `*-prompt.md` 模板文件（implementer、task-reviewer、code-reviewer 等） | 填充模板，然后使用 `agent_name: "generalist"` 调用 `invoke_agent`，并传入填充后的提示词 |
| 引用 `superpowers:requesting-code-review` 的 `./code-reviewer.md` | 使用 `agent_name: "generalist"` 调用 `invoke_agent`，并传入填充后的评审模板 |
| 内联提示词（未引用模板） | 使用 `agent_name: "generalist"` 调用 `invoke_agent`，并传入内联提示词 |

### 填充提示词

技能会提供带有 `{WHAT_WAS_IMPLEMENTED}` 或 `[FULL TEXT of task]` 等占位符的提示词模板。在传给 `invoke_agent` 之前，填充所有占位符，并传入完整提示词。提示词模板本身包含代理的角色、评审标准和预期输出格式——子代理会遵循这些要求。

### 并行调度

Gemini CLI 支持并行调度子代理。在同一响应中发起多个 `invoke_agent` 调用（或在一个提示中发起多个 `@generalist` 调用）。保持有依赖关系的任务按顺序执行，但不要仅为了让历史记录更简单，就把彼此独立的任务串行化。

## Gemini CLI 的其他工具

| 工具 | 用途 |
|------|------|
| `save_memory`（旧版） | 在 `experimental.memoryV2 = false` 时持久化跨会话事实 |
| `get_internal_docs` | 查询 Gemini CLI 内置文档 |
| `ask_user` | 提出结构化问题（文本/单选/多选） |
| `enter_plan_mode` / `exit_plan_mode` | 进入或退出只读计划模式 |
| `update_topic` | 更新当前对话的主题/战略意图元数据 |
| `complete_task` | 发出 Gemini 子代理已完成的信号，并将其结果返回给父代理 |
| `tracker_create_task`、`tracker_update_task`、`tracker_get_task`、`tracker_list_tasks`、`tracker_add_dependency`、`tracker_visualize` | 丰富的任务跟踪器，支持依赖关系 |
| `read_mcp_resource`、`list_mcp_resources` | 访问 MCP 资源 |
