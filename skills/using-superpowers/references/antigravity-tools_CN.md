# Antigravity CLI（`agy`）工具映射

技能会用行动来表述需求（“派发一个子 agent”、“创建待办事项”、“读取文件”）。在 Antigravity CLI（`agy`）中，这些行动对应以下工具。

| 技能请求的行动 | Antigravity CLI 等价操作 |
|----------------------|----------------------|
| 派发一个子 agent（`Subagent (general-purpose):` 模板） | 使用内置的 `TypeName` 调用 `invoke_subagent`——完整能力工作使用 `self`，只读工作使用 `research` |
| 任务跟踪（“创建待办事项”、“标记完成”） | 一个**任务工件**——使用 `write_to_file`，并设置 `IsArtifact: true` 及 `ArtifactType: "task"`（见[任务跟踪](#任务跟踪)）。不要使用 `manage_task`，它管理后台进程。 |

## 任务跟踪

Antigravity 没有待办工具（`manage_task` 管理后台进程——`list`/`kill`/`status`/`send_input`——它不是检查清单）。当技能要求创建待办列表或跟踪任务时，维护一个任务工件：使用 `write_to_file` 保存 Markdown 检查清单（`IsArtifact: true`、`ArtifactMetadata.ArtifactType: "task"`），并使用 `replace_file_content` / `multi_replace_file_content` 随进度编辑它。

在任何多步骤任务开始时，创建任务工件，列出计划中的每一步。每完成一步，就编辑工件将其标记为完成（`- [x]`）。如果计划发生变化，更新检查清单。保持它最新——它是你唯一的事实来源；对话变长后，在开始每一步之前重新读取它。
