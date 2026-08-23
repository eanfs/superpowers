# Hermes Agent 工具映射

技能以动作（“调度子代理”“创建待办事项”“读取文件”）进行描述。在 Hermes Agent 上，这些动作对应以下工具。

## 工具

| 技能请求的动作 | Hermes 工具 |
|---|---|
| 读取文件 | `read_file` |
| 创建新文件 | `write_file` |
| 编辑文件（定向补丁） | `patch` |
| 运行 Shell 命令 | `terminal` |
| 搜索文件内容 | `search_files` |
| 按名称查找文件 | 使用 `find` 的 `terminal` |
| 获取 URL / 读取网页 | `web_extract(urls=[...])` |
| 搜索网页 | `web_search(query=...)` |
| 调度子代理 | `delegate_task(goal=..., context=..., toolsets=[...], role="leaf")` |
| 任务跟踪 | `todo` 工具 |
| 调用技能 | `skill_view("skill-name")` |

## 指令文件

当技能提到“你的指令文件”时，在项目目录中指 **`AGENTS.md`**，或全局目录 `~/.hermes/SOUL.md` 中的 **`SOUL.md`**。

## 调用技能

Hermes Agent 提供带有 `skill_view` 和 `skills_list` 工具的 `skills` 工具集。

要调用 Superpowers 技能，请使用：

```
skill_view("brainstorming")
skill_view("test-driven-development")
```

如果 `skill_view` 找不到 Superpowers 技能（技能可能要等插件完全注册后才出现在目录中），则直接读取 `SKILL.md`：

```
read_file(path="~/.hermes/plugins/superpowers/skills/<skill-name>/SKILL.md")
```

这种回退机制与其他不具备原生技能加载能力的运行时所使用的机制相同。

## 子代理调度

使用 `delegate_task` 生成隔离的子代理工作流：

```
delegate_task(goal="...", context="...", toolsets=[...], role="leaf")
```

如果 `delegate_task` 不可用，则在当前会话中按顺序执行工作，不要臆造工具调用。

## 任务跟踪

Hermes Agent 会话内使用 `todo` 工具进行任务跟踪。对于多代理任务看板，如果可用，使用 `hermes kanban` CLI。将旧版 Superpowers 文档提到的 `TodoWrite` 视为上述任务跟踪动作。
