# 可视化伴侣指南

用于展示 mockup、图表和选项的基于浏览器的可视化头脑风暴伴侣。

## 何时使用

按问题决定，而不是按会话决定。判断标准：**用户通过观看是否会比阅读更好地理解？**

**当内容本身是可视化内容时使用浏览器：**

- **UI mockup**——线框图、布局、导航结构、组件设计
- **架构图**——系统组件、数据流、关系图
- **并排可视化比较**——比较两种布局、两种配色方案、两种设计方向
- **设计润色**——问题涉及外观感受、间距、视觉层次时
- **空间关系**——以图表形式呈现的状态机、流程图、实体关系

**当内容是文本或表格时使用终端：**

- **需求和范围问题**——“X 是什么意思？”、“哪些功能在范围内？”
- **概念性的 A/B/C 选择**——在用文字描述的方案之间做选择
- **权衡列表**——优点/缺点、比较表
- **技术决策**——API 设计、数据建模、架构方案选择
- **澄清问题**——答案是文字，而不是视觉偏好的任何问题

关于 UI 主题的问题不一定就是可视化问题。“你想要什么类型的向导？”是概念性问题——使用终端。“这些向导布局中你觉得哪个更合适？”是可视化问题——使用浏览器。

## 工作原理

服务器监视一个包含 HTML 文件的目录，并将最新文件提供给浏览器。你把 HTML 内容写入 `screen_dir`，用户会在浏览器中看到它，并可以点击选择选项。选择结果会记录到 `state_dir/events`，你在下一轮读取该文件。

**内容片段与完整文档：**如果 HTML 文件以 `<!DOCTYPE` 或 `<html` 开头，服务器会按原样提供它（只注入辅助脚本）。否则，服务器会自动将你的内容包装进框架模板——添加页眉、CSS 主题、连接状态和全部交互基础设施。**默认编写内容片段。**只有在需要完全控制页面时，才编写完整文档。

## 启动会话

```bash
# Start AFTER the user approves the companion. --open auto-opens their browser on
# the first screen; --project-dir persists mockups and enables same-port restart.
scripts/start-server.sh --project-dir /path/to/project --open

# Returns: {"type":"server-started","port":52341,
#           "url":"http://localhost:52341/?key=ab12…",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#           "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

从响应中保存 `screen_dir` 和 `state_dir`。使用 `--open` 时，当你推送第一个界面，浏览器会自动打开——无需要求用户手动打开，但仍应分享 URL 作为备用方案（无头/远程环境不会自动打开）。

**URL 包含会话密钥（`?key=…`）。**服务器会拒绝不带密钥的请求，因此始终向用户提供 `url` 字段中的**完整** URL——绝不要去掉查询字符串，也绝不要提供不带密钥的 `http://host:port`。密钥控制 HTTP 和 WebSocket 访问权限，防止无关的浏览器标签页或网络中的其他机器读取界面或注入事件。首次加载后，浏览器会通过 cookie 记住密钥，因此重新加载和访问 `/files/*` 资源时无需再次提供。

**查找连接信息：**服务器会将启动 JSON 写入 `$STATE_DIR/server-info`。如果你在后台启动服务器时没有捕获 stdout，请读取该文件来获取 URL 和端口。使用 `--project-dir` 时，检查 `<project>/.superpowers/brainstorm/` 中的会话目录。

**注意：**将项目根目录作为 `--project-dir` 传入，这样 mockup 会持久化在 `.superpowers/brainstorm/` 中，并在服务器重启后保留。否则文件会写入 `/tmp` 并被清理。如果项目尚未配置，请提醒用户将 `.superpowers/` 加入 `.gitignore`。

**按平台启动服务器：**

**Claude Code：**
```bash
# Default mode works — the script backgrounds the server itself.
scripts/start-server.sh --project-dir /path/to/project --open
```

在 Windows 上，脚本会自动检测并切换到前台模式（这会阻塞工具调用）。在 Bash 工具调用中使用 `run_in_background: true`，以便服务器跨会话轮次继续运行，然后在下一轮读取 `$STATE_DIR/server-info` 来获取 URL 和端口。

**Codex：**
```bash
# Codex reaps background processes. The script auto-detects CODEX_CI and
# switches to foreground mode. Run it normally — no extra flags needed.
scripts/start-server.sh --project-dir /path/to/project --open
```

**Gemini CLI：**
```bash
# Use --foreground and set is_background: true on your shell tool call
# so the process survives across turns
scripts/start-server.sh --project-dir /path/to/project --open --foreground
```

**Copilot CLI：**
```bash
# Start it with Copilot CLI's non-blocking/background shell mechanism so the
# server survives across turns. Keep --foreground so the harness, not the
# script, owns backgrounding. The launcher is a .sh, so invoke it via bash
# (on Windows, call Git Bash's bash.exe from the PowerShell tool).
bash scripts/start-server.sh --project-dir /path/to/project --open --foreground
```

**其他环境：**服务器必须在会话轮次之间持续在后台运行。如果你的环境会回收脱离的进程，请使用 `--foreground`，并通过平台提供的后台执行机制启动命令。

如果浏览器无法访问该 URL（远程/容器环境中很常见），请绑定到非回环主机：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

使用 `--url-host` 控制返回的 URL JSON 中打印的主机名。

## 循环流程

1. **检查服务器是否存活**，然后在 `screen_dir` 中创建新文件并**写入 HTML**：
   - **必须在引用 URL 或推送界面之前确认服务器存活。**检查 `$STATE_DIR/server-info` 是否存在，以及 `$STATE_DIR/server-stopped` 是否不存在。如果服务器已关闭，使用**相同的 `--project-dir`** 通过 `start-server.sh` 重启——它会复用相同端口，因此用户打开的标签页会自行重新连接（服务器关闭时会显示“已暂停”遮罩），无需发送新的 URL。服务器空闲 4 小时后会自动退出（可通过 `--idle-timeout-minutes` 配置）。
   - 使用语义化文件名：`platform.html`、`visual-style.html`、`layout.html`
   - **绝不要复用文件名**——每个界面都必须使用新文件
   - 使用文件创建工具——**绝不要使用 cat/heredoc**（会把噪声倾倒到终端）
   - 服务器会自动提供最新文件

2. **告诉用户应看到什么，然后结束本轮：**
   - 提醒用户 URL（每一步都要提醒，而不只是第一次）
   - 简要说明界面上显示的内容（例如：“正在展示首页的 3 个布局选项”）
   - 请用户在终端回复：“看一下并告诉我你的想法。如果愿意，可以点击选择一个选项。”

3. **下一轮**——用户在终端回复后：
   - 如果 `$STATE_DIR/events` 存在，读取它——其中每行是一个 JSON 对象，记录用户在浏览器中的交互（点击、选择）
   - 将用户的终端文本与其合并，以获得完整信息
   - 终端消息是主要反馈；`state_dir/events` 提供结构化交互数据

4. **迭代或继续**——如果反馈改变了当前界面，写入新文件（例如 `layout-v2.html`）。只有当前步骤得到验证后，才进入下一个问题。

5. **返回终端时卸载界面**——当下一步不需要浏览器时（例如澄清问题、权衡讨论），推送一个等待界面来清除过时内容：

   ```html
   <!-- filename: waiting.html (or waiting-2.html, etc.) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   这样可以避免用户在对话已经继续后仍盯着已经解决的选择。出现下一个可视化问题时，照常推送新的内容文件。

6. 重复以上流程，直到完成。

## 编写内容片段

只编写页面内部所需的内容。服务器会自动将它包装进框架模板（页眉、主题 CSS、连接状态和全部交互基础设施）。

**最小示例：**

```html
<h2>Which layout works better?</h2>
<p class="subtitle">Consider readability and visual hierarchy</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single Column</h3>
      <p>Clean, focused reading experience</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Two Column</h3>
      <p>Sidebar navigation with main content</p>
    </div>
  </div>
</div>
```

就是这样。不需要 `<html>`、CSS 或 `<script>` 标签。服务器会提供所有这些内容。

## 可用的 CSS 类

框架模板为你的内容提供以下 CSS 类：

### 选项（A/B/C 选择）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Title</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

**多选：**向容器添加 `data-multiselect`，即可允许用户选择多个选项。每次点击都会切换该项目的选中样式。

```html
<div class="options" data-multiselect>
  <!-- same option markup — users can select/deselect multiple -->
</div>
```

### 卡片（视觉设计）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup content --></div>
    <div class="card-body">
      <h3>Name</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

### Mockup 容器

```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard Layout</div>
  <div class="mockup-body"><!-- your mockup HTML --></div>
</div>
```

### 分栏视图（并排）

```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### 优点/缺点

```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>Benefit</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>Drawback</li></ul></div>
</div>
```

### Mock 元素（线框图构建块）

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">Navigation</div>
  <div class="mock-content">Main content area</div>
</div>
<button class="mock-button">Action Button</button>
<input class="mock-input" placeholder="Input field">
<div class="placeholder">Placeholder area</div>
```

### 排版与章节

- `h2`——页面标题
- `h3`——章节标题
- `.subtitle`——标题下方的辅助文字
- `.section`——带有底部外边距的内容块
- `.label`——小号大写标签文字

## 浏览器事件格式

当用户在浏览器中点击选项时，其交互会记录到 `$STATE_DIR/events`（每行一个 JSON 对象）。推送新界面时，该文件会自动清空。

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

完整的事件流会展示用户的探索路径——他们可能会在确定之前点击多个选项。最后一个 `choice` 事件通常是最终选择，但点击模式也能揭示值得进一步询问的犹豫或偏好。

如果 `$STATE_DIR/events` 不存在，说明用户没有与浏览器交互——只使用他们的终端文本。

## 设计提示

- **根据问题调整保真度**——布局问题使用线框图，润色问题使用精致设计
- **在每个页面解释问题**——使用“哪个布局看起来更专业？”而不只是“选一个”
- **继续之前先迭代**——如果反馈改变了当前界面，写入新版本
- **每个界面最多 2–4 个选项**
- **在实际内容重要时使用真实内容**——对于摄影作品集，使用实际图片（Unsplash）。占位内容会掩盖设计问题。
- **保持 mockup 简单**——关注布局和结构，而不是像素级完美设计

## 文件命名

- 使用语义化名称：`platform.html`、`visual-style.html`、`layout.html`
- 绝不要复用文件名——每个界面都必须使用新文件
- 迭代时：添加版本后缀，例如 `layout-v2.html`、`layout-v3.html`
- 服务器按修改时间提供最新文件

## 清理

```bash
scripts/stop-server.sh $SESSION_DIR
```

如果会话使用了 `--project-dir`，mockup 文件会保留在 `.superpowers/brainstorm/` 中供日后参考。只有 `/tmp` 会话会在停止时被删除。

## 参考

- 框架模板（CSS 参考）：`scripts/frame-template.html`
- 辅助脚本（客户端）：`scripts/helper.js`
