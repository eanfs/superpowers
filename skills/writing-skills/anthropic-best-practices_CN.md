# 技能编写最佳实践

> 学习如何编写有效的技能，使代理能够成功发现并使用它们。

优秀的技能简洁、结构清晰，并经过真实使用测试。本指南提供实用的编写决策，帮助你编写代理能够发现并有效使用的技能。

关于技能工作方式的概念背景，参见 [Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)。

## 核心原则

### 简洁是关键

[上下文窗口](https://platform.claude.com/docs/en/build-with-claude/context-windows)是一种公共资源。你的技能会与代理所需的其他所有内容共享上下文窗口，包括：

* 系统提示词
* 对话历史
* 其他技能的元数据
* 你的实际请求

并不是技能中的每个 token 都会立即产生成本。启动时，只有所有技能的元数据（名称和描述）会预先加载。只有技能变得相关时，代理才会读取 `SKILL.md`，也只会在需要时读取其他文件。不过，保持 `SKILL.md` 简洁仍然很重要：代理加载它后，每个 token 都会与对话历史和其他上下文竞争。

**默认假设：**代理已经非常聪明

只添加代理不知道的上下文。逐项质疑信息：

* “代理真的需要这个解释吗？”
* “我能假设代理知道这一点吗？”
* “这段文字是否值得它占用的 token？”

**优秀示例：简洁**（约 50 个 token）：

````markdown  theme={null}
## Extract PDF text

Use pdfplumber for text extraction:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**糟糕示例：过于冗长**（约 150 个 token）：

```markdown  theme={null}
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but we
recommend pdfplumber because it's easy to use and handles most cases well.
First, you'll need to install it using pip. Then you can use the code below...
```

简洁版本假设代理知道 PDF 是什么，也知道库如何工作。

### 设置合适的自由度

让具体程度匹配任务的脆弱性和可变性。

**高自由度**（基于文本的指令）：

适用于：

* 有多种有效方法
* 决策取决于上下文
* 启发式方法指导处理

示例：

```markdown  theme={null}
## Code review process

1. Analyze the code structure and organization
2. Check for potential bugs or edge cases
3. Suggest improvements for readability and maintainability
4. Verify adherence to project conventions
```

**中等自由度**（带参数的伪代码或脚本）：

适用于：

* 存在首选模式
* 允许一定变化
* 配置会影响行为

示例：

````markdown  theme={null}
## Generate report

Use this template and customize as needed:

```python
def generate_report(data, format="markdown", include_charts=True):
    # Process data
    # Generate output in specified format
    # Optionally include visualizations
```
````

**低自由度**（具体脚本，参数很少或没有参数）：

适用于：

* 操作脆弱且容易出错
* 一致性至关重要
* 必须遵循特定顺序

示例：

````markdown  theme={null}
## Database migration

Run exactly this script:

```bash
python scripts/migrate.py --verify --backup
```

Do not modify the command or add additional flags.
````

**类比：**把代理想象成沿着一条道路探索的机器人：

* **两侧都是悬崖的窄桥：**只有一种安全前进的方式。提供具体护栏和精确指令（低自由度）。例如，必须按精确顺序运行的数据库迁移。
* **没有危险的开阔地：**有很多路径都能成功。给出总体方向，相信代理会找到最佳路线（高自由度）。例如，最佳方法取决于上下文的代码评审。

### 用计划使用的所有模型进行测试

技能是模型的附加内容，因此效果取决于底层模型。用计划使用的所有模型测试技能。

**按模型考虑的测试点：**

* **Claude Haiku**（快速、经济）：技能是否提供了足够指引？
* **Claude Sonnet**（均衡）：技能是否清晰高效？
* **Claude Opus**（强大推理）：技能是否避免过度解释？

对 Opus 完美有效的内容，对 Haiku 可能需要更多细节。如果计划在多个模型上使用技能，应力求对所有模型都有效。

## 技能结构

<Note>
  **YAML Frontmatter：**SKILL.md 的 frontmatter 需要两个字段：

  * `name` - 技能的人类可读名称（最多 64 个字符）
  * `description` - 对技能作用和使用时机的一行描述（最多 1024 个字符）

  关于完整的技能结构细节，参见 [Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#skill-structure)。
</Note>

### 命名约定

使用一致的命名模式，方便引用和讨论技能。我们建议使用**动名词形式**（动词 + -ing）作为技能名称，这能清晰描述技能提供的活动或能力。

**优秀的命名示例（动名词形式）：**

* “Processing PDFs”
* “Analyzing spreadsheets”
* “Managing databases”
* “Testing code”
* “Writing documentation”

**可接受的替代形式：**

* 名词短语：“PDF Processing”“Spreadsheet Analysis”
* 面向动作的形式：“Process PDFs”“Analyze Spreadsheets”

**避免：**

* 模糊名称：“Helper”“Utils”“Tools”
* 过于通用：“Documents”“Data”“Files”
* 技能集合内部模式不一致

一致的命名有助于：

* 在文档和对话中引用技能
* 一眼理解技能的作用
* 组织和搜索技能
* 维护专业、统一的技能库

### 编写有效的描述

`description` 字段支持技能发现，应同时包含技能做什么以及何时使用。

<Warning>
  **始终使用第三人称。**描述会被注入系统提示词，不一致的视角可能导致发现问题。

  * **推荐：**“Processes Excel files and generates reports”
  * **避免：**“I can help you process Excel files”
  * **避免：**“You can use this to process Excel files”
</Warning>

**要具体并包含关键术语。**同时说明技能做什么，以及使用技能的具体触发条件/上下文。

每个技能只有一个 description 字段。描述对于技能选择至关重要：代理会从可能超过 100 个技能中选择合适的技能，必须凭描述判断是否选择。`SKILL.md` 的其余部分再提供实现细节。

有效示例：

**PDF 处理技能：**

```yaml  theme={null}
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel 分析技能：**

```yaml  theme={null}
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git 提交助手技能：**

```yaml  theme={null}
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

避免使用以下模糊描述：

```yaml  theme={null}
description: Helps with documents
```

```yaml  theme={null}
description: Processes data
```

```yaml  theme={null}
description: Does stuff with files
```

### 渐进式披露模式

SKILL.md 充当概览，像入门指南的目录一样指向详细材料。关于渐进式披露的解释，参见概览中的 [How Skills work](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)。

**实用指引：**

* 为获得最佳性能，保持 SKILL.md 正文少于 500 行
* 接近此限制时，将内容拆分到单独文件
* 使用下面的模式组织指令、代码和资源

#### 可视化概览：从简单到复杂

一个基础技能只需一个包含元数据和指令的 SKILL.md 文件：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=87782ff239b297d9a9e8e1b72ed72db9" alt="显示 YAML frontmatter 和 Markdown 正文的简单 SKILL.md 文件" data-og-width="2048" width="2048" data-og-height="1153" height="1153" data-path="images/agent-skills-simple-file.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=c61cc33b6f5855809907f7fda94cd80e 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=90d2c0c1c76b36e8d485f49e0810dbfd 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=ad17d231ac7b0bea7e5b4d58fb4aeabb 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f5d0a7a3c668435bb0aee9a3a8f8c329 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0e927c1af9de5799cfe557d12249f6e6 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=46bbb1a51dd4c8202a470ac8c80a893d 2500w" />

随着技能增长，可以打包只在需要时加载的附加内容：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=a5e0aa41e3d53985a7e3e43668a33ea3" alt="将 reference.md 和 forms.md 等附加参考文件打包在一起" data-og-width="2048" width="2048" data-og-height="1327" height="1327" data-path="images/agent-skills-bundling-content.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f8a0e73783e99b4a643d79eac86b70a2 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=dc510a2a9d3f14359416b706f067904a 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=82cd6286c966303f7dd914c28170e385 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=56f3be36c77e4fe4b523df209a6824c6 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=d22b5161b2075656417d56f41a74f3dd 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=3dd4bdd6850ffcc96c6c45fcb0acd6eb 2500w" />

完整的技能目录结构可能如下：

```
pdf/
├── SKILL.md              # Main instructions (loaded when triggered)
├── FORMS.md              # Form-filling guide (loaded as needed)
├── reference.md          # API reference (loaded as needed)
├── examples.md           # Usage examples (loaded as needed)
└── scripts/
    ├── analyze_form.py   # Utility script (executed, not loaded)
    ├── fill_form.py      # Form filling script
    └── validate.py       # Validation script
```

#### 模式 1：带参考文件的高层指南

````markdown  theme={null}
# PDF Processing

## Quick start

Extract text with pdfplumber:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced features

**Form filling**: See [FORMS.md](FORMS.md) for complete guide
**API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
**Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
````

代理只会在需要时读取 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

#### 模式 2：按领域组织

对于包含多个领域的技能，按领域组织内容，避免加载无关上下文。当用户询问销售指标时，代理只需要读取销售相关的模式，而不是财务或营销数据。这样可以保持 token 使用量低，并让上下文聚焦。

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

````markdown SKILL.md theme={null}
# BigQuery Data Analysis

## Available datasets

**Finance**: Revenue, ARR, billing → See [reference/finance.md](reference/finance.md)
**Sales**: Opportunities, pipeline, accounts → See [reference/sales.md](reference/sales.md)
**Product**: API usage, features, adoption → See [reference/product.md](reference/product.md)
**Marketing**: Campaigns, attribution, email → See [reference/marketing.md](reference/marketing.md)

## Quick search

Find specific metrics using grep:

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
````

#### 模式 3：条件式细节

展示基础内容，并链接到高级内容：

```markdown  theme={null}
# DOCX Processing

## Creating documents

Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

只有当用户需要这些功能时，代理才会读取 REDLINING.md 或 OOXML.md。

### 避免嵌套过深的参考文件

当代理从其他被引用的文件中遇到嵌套引用时，可能只读取文件的一部分。代理可能会使用 `head -100` 等命令预览内容，而不是读取完整文件，导致信息不完整。

**让从 SKILL.md 出发的参考文件保持一层深度。**所有参考文件都应由 SKILL.md 直接链接，以确保需要时能完整读取。

**糟糕示例：嵌套过深：**

```markdown  theme={null}
# SKILL.md
See [advanced.md](advanced.md)...

# advanced.md
See [details.md](details.md)...

# details.md
Here's the actual information...
```

**优秀示例：一层深度：**

```markdown  theme={null}
# SKILL.md

**Basic usage**: [instructions in SKILL.md]
**Advanced features**: See [advanced.md](advanced.md)
**API reference**: See [reference.md](reference.md)
**Examples**: See [examples.md](examples.md)
```

### 为较长的参考文件设置目录

对于超过 100 行的参考文件，在顶部加入目录。这确保代理即使只通过部分读取预览文件，也能看到全部范围。

**示例：**

```markdown  theme={null}
# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...

## Core methods
...
```

关于这种基于文件系统的架构如何实现渐进式披露，参见高级部分的[运行时环境](#runtime-environment)章节。

## 工作流与反馈循环

### 为复杂任务使用工作流

将复杂操作拆分为清晰的连续步骤。对于特别复杂的工作流，提供代理可以复制到响应中并在过程中勾选的清单。

**示例 1：研究综合工作流**（适用于无代码技能）：

````markdown  theme={null}
## Research synthesis workflow

Copy this checklist and track your progress:

```
Research Progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**

Review each document in the `sources/` directory. Note the main arguments and supporting evidence.

**Step 2: Identify key themes**

Look for patterns across sources. What themes appear repeatedly? Where do sources agree or disagree?

**Step 3: Cross-reference claims**

For each major claim, verify it appears in the source material. Note which source supports each point.

**Step 4: Create structured summary**

Organize findings by theme. Include:
- Main claim
- Supporting evidence from sources
- Conflicting viewpoints (if any)

**Step 5: Verify citations**

Check that every claim references the correct source document. If citations are incomplete, return to Step 3.
````

这个示例展示了工作流如何应用于不需要代码的分析任务。清单模式适用于任何复杂的多步骤任务。

**示例 2：PDF 表单填写工作流**（适用于带代码的技能）：

````markdown  theme={null}
## PDF form filling workflow

Copy this checklist and check off items as you complete them:

```
Task Progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify output (run verify_output.py)
```

**Step 1: Analyze the form**

Run: `python scripts/analyze_form.py input.pdf`

This extracts form fields and their locations, saving to `fields.json`.

**Step 2: Create field mapping**

Edit `fields.json` to add values for each field.

**Step 3: Validate mapping**

Run: `python scripts/validate_fields.py fields.json`

Fix any validation errors before continuing.

**Step 4: Fill the form**

Run: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify output**

Run: `python scripts/verify_output.py output.pdf`

If verification fails, return to Step 2.
````

清晰的步骤可以防止代理跳过关键验证。清单能帮助你和代理跟踪多步骤工作流的进度。

### 实现反馈循环

**常见模式：**运行验证器 → 修复错误 → 重复

这个模式能显著提高输出质量。

**示例 1：样式指南合规**（适用于无代码技能）：

```markdown  theme={null}
## Content review process

1. Draft your content following the guidelines in STYLE_GUIDE.md
2. Review against the checklist:
   - Check terminology consistency
   - Verify examples follow the standard format
   - Confirm all required sections are present
3. If issues found:
   - Note each issue with specific section reference
   - Revise the content
   - Review the checklist again
4. Only proceed when all requirements are met
5. Finalize and save the document
```

这个示例展示了使用参考文件而不是脚本的验证循环模式。“验证器”是 STYLE_GUIDE.md，代理通过读取并对比来执行检查。

**示例 2：文档编辑流程**（适用于带代码的技能）：

```markdown  theme={null}
## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues in the XML
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. Test the output document
```

验证循环可以尽早捕获错误。

## 内容指南

### 避免时间敏感信息

不要包含会过时的信息：

**糟糕示例：时间敏感**（会变错）：

```markdown  theme={null}
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
```

**优秀示例**（使用“旧模式”章节）：

```markdown  theme={null}
## Current method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>

The v1 API used: `api.example.com/v1/messages`

This endpoint is no longer supported.
</details>
```

旧模式章节提供历史背景，而不会让主要内容变得杂乱。

### 使用一致的术语

选择一个术语，并在整个技能中使用它：

**优秀——一致：**

* 始终使用“API endpoint”
* 始终使用“field”
* 始终使用“extract”

**糟糕——不一致：**

* 混用“API endpoint”“URL”“API route”“path”
* 混用“field”“box”“element”“control”
* 混用“extract”“pull”“get”“retrieve”

一致性有助于代理理解并遵循指令。

## 常见模式

### 模板模式

为输出格式提供模板。让严格程度匹配你的需求。

**严格要求**（例如 API 响应或数据格式）：

````markdown  theme={null}
## Report structure

ALWAYS use this exact template structure:

```markdown
# [Analysis Title]

## Executive summary
[One-paragraph overview of key findings]

## Key findings
- Finding 1 with supporting data
- Finding 2 with supporting data
- Finding 3 with supporting data

## Recommendations
1. Specific actionable recommendation
2. Specific actionable recommendation
```
````

**灵活指引**（适合需要调整的情况）：

````markdown  theme={null}
## Report structure

Here is a sensible default format, but use your best judgment based on the analysis:

```markdown
# [Analysis Title]

## Executive summary
[Overview]

## Key findings
[Adapt sections based on what you discover]

## Recommendations
[Tailor to the specific context]
```

Adjust sections as needed for the specific analysis type.
````

### 示例模式

对于输出质量取决于示例的技能，像常规提示词一样提供输入/输出对：

````markdown  theme={null}
## Commit message format

Generate commit messages following these examples:

**Example 1:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**Example 2:**
Input: Fixed bug where dates displayed incorrectly in reports
Output:
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

**Example 3:**
Input: Updated dependencies and refactored error handling
Output:
```
chore: update dependencies and refactor error handling

- Upgrade lodash to 4.17.21
- Standardize error response format across endpoints
```

Follow this style: type(scope): brief description, then detailed explanation.
````

示例比单纯的描述更能帮助代理理解目标风格和细节程度。

### 条件式工作流模式

引导代理经过决策点：

```markdown  theme={null}
## Document modification workflow

1. Determine the modification type:

   **Creating new content?** → Follow "Creation workflow" below
   **Editing existing content?** → Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
   - Repack when complete
```

<Tip>
  如果工作流变得庞大、复杂且包含许多步骤，可以考虑将其移到单独文件，并告诉代理根据当前任务读取相应文件。
</Tip>

## 评估与迭代

### 先构建评估

**在编写大量文档之前创建评估。**这能确保你的技能解决真实问题，而不是记录想象中的问题。

**评估驱动开发：**

1. **找出缺口：**在没有技能的情况下让代理处理有代表性的任务。记录具体失败或缺失的上下文
2. **创建评估：**构建三个测试这些缺口的场景
3. **建立基线：**测量没有技能时代理的表现
4. **编写最小指令：**只创建足以处理缺口并通过评估的内容
5. **迭代：**执行评估，对比基线并改进

这种方法确保你在解决真实问题，而不是预先考虑可能永远不会出现的需求。

**评估结构：**

```json  theme={null}
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF file using an appropriate PDF processing library or command-line tool",
    "Extracts text content from all pages of the document without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
  ]
}
```

<Note>
  此示例演示了一个简单测试规范的数据驱动评估。我们目前没有提供运行这些评估的内置方式。用户可以创建自己的评估系统。评估结果是衡量技能效果的事实依据。
</Note>

### 与代理一起迭代开发技能

最有效的技能开发流程包含代理本身。与一个实例（“代理 A”）合作创建技能，供其他实例（“代理 B”）使用。代理 A 帮助你设计和改进指令，代理 B 在真实任务中测试指令。这样做有效，是因为底层模型既理解如何编写有效的代理指令，也理解代理需要什么信息。

**创建新技能：**

1. **在没有技能的情况下完成任务：**使用普通提示词与代理 A 解决问题。在工作过程中，你会自然提供上下文、解释偏好并分享流程知识。注意哪些信息被你反复提供。

2. **找出可复用模式：**完成任务后，确定你提供的哪些上下文对类似未来任务有用。

   **示例：**如果你完成了一次 BigQuery 分析，可能提供了表名、字段定义、筛选规则（例如“始终排除测试账户”）以及常见查询模式。

3. **让代理 A 创建技能：**“创建一个技能，记录我们刚才使用的 BigQuery 分析模式。包含表结构、命名约定，以及筛选测试账户的规则。”

   <Tip>
     现代代理原生理解技能格式和结构。你不需要特殊系统提示词或“编写技能”技能来帮助创建技能。直接让代理创建技能，它就会生成结构正确的 SKILL.md 内容，包括合适的 frontmatter 和正文。
   </Tip>

4. **检查简洁性：**检查代理 A 是否添加了不必要的解释。询问：“删除关于胜率含义的解释——代理已经知道它。”

5. **改进信息架构：**让代理 A 更有效地组织内容。例如：“把表结构组织到单独的参考文件中。以后可能还会增加更多表。”

6. **在类似任务上测试：**使用代理 B（一个加载了技能的新实例）处理相关用例。观察代理 B 是否找到正确的信息、正确应用规则并成功完成任务。

7. **根据观察迭代：**如果代理 B 遇到困难或遗漏某点，带着具体情况回到代理 A：“代理使用这个技能时，做 Q4 查询忘记按日期筛选。技能提到了这条规则，但也许应该更突出日期筛选模式？”

**迭代现有技能：**

改进技能时同样遵循上述层级模式。你在以下工作之间交替：

* **与代理 A 合作**（帮助改进技能的专家）
* **使用代理 B 测试**（使用已加载技能的代理）
* **观察代理 B 的行为**并将洞察带回代理 A

1. **在真实工作流中使用技能：**给加载了技能的代理 B 实际任务，而不是测试场景

2. **观察代理 B 的行为：**注意它在哪里遇到困难、成功，或做出意外选择

   **观察示例：**“我让代理 B 生成区域销售报告时，它写了查询，却忘记排除测试账户，尽管技能提到了这条规则。”

3. **回到代理 A 改进：**分享当前的 SKILL.md 并描述观察结果。询问：“我注意到让代理 B 生成区域报告时忘记筛选测试账户。技能提到了筛选，也许应该让规则更突出？”

4. **评审代理 A 的建议：**代理 A 可能建议重新组织内容以突出规则，使用“必须筛选”这类更强的语言，而不是“始终筛选”，或重构工作流章节。

5. **应用并测试改动：**使用代理 A 的改进更新技能，然后在类似请求上再次用代理 B 测试

6. **根据使用情况重复：**遇到新场景时继续观察—改进—测试循环。每次迭代都基于真实代理行为而非假设来改进技能。

**收集团队反馈：**

1. 与团队成员分享技能并观察他们的使用方式
2. 询问：技能是否在预期时机激活？指令是否清晰？缺少什么？
3. 纳入反馈，处理自己使用模式中的盲点

**这种方法有效的原因：**代理 A 理解代理的需求，你提供领域专业知识，代理 B 通过真实使用揭示缺口，迭代改进则基于观察而非假设。

### 观察代理如何浏览技能

迭代技能时，关注代理实际如何使用它们。留意：

* **意外的探索路径：**代理是否按你没预料的顺序读取文件？这可能说明你的结构不像你以为的那么直观
* **遗漏的连接：**代理是否没有跟随指向重要文件的引用？你的链接可能需要更明确或更突出
* **过度依赖某些章节：**如果代理反复读取同一个文件，考虑这些内容是否应放入主 SKILL.md
* **被忽略的内容：**如果代理从未访问某个打包文件，它可能是不必要的，或者主指令中的提示不够明显

根据这些观察而不是假设进行迭代。技能元数据中的 `name` 和 `description` 尤其关键。代理会依据它们判断是否针对当前请求触发技能。确保它们清晰说明技能做什么以及何时使用。

## 应避免的反模式

### 避免 Windows 风格路径

即使在 Windows 上，也始终使用正斜杠表示文件路径：

* ✓ **推荐：**`scripts/helper.py`、`reference/guide.md`
* ✗ **避免：**`scripts\helper.py`、`reference\guide.md`

Unix 风格路径跨平台工作，而 Windows 风格路径会在 Unix 上导致错误。

### 避免提供过多选项

除非必要，不要提供多种方法：

````markdown  theme={null}
**Bad example: Too many choices** (confusing):
"You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or..."

**Good example: Provide a default** (with escape hatch):
"Use pdfplumber for text extraction:
```python
import pdfplumber
```

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."
````

## 高级：带可执行代码的技能

下面的章节关注包含可执行脚本的技能。如果你的技能只包含 Markdown 指令，请跳到[有效技能清单](#checklist-for-effective-skills)。

### 解决问题，不要推给代理

编写技能脚本时，处理错误条件，而不是把问题推给代理。

**优秀示例：明确处理错误：**

```python  theme={null}
def process_file(path):
    """Process a file, creating it if it doesn't exist."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # Create file with default content instead of failing
        print(f"File {path} not found, creating default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # Provide alternative instead of failing
        print(f"Cannot access {path}, using default")
        return ''
```

**糟糕示例：推给代理：**

```python  theme={null}
def process_file(path):
    # Just fail and let the agent figure it out
    return open(path).read()
```

还应说明配置参数的理由，避免“巫术常量”（Ousterhout 定律）。如果你不知道某个值为什么合适，代理又如何确定它？

**优秀示例：自描述：**

```python  theme={null}
# HTTP requests typically complete within 30 seconds
# Longer timeout accounts for slow connections
REQUEST_TIMEOUT = 30

# Three retries balances reliability vs speed
# Most intermittent failures resolve by the second retry
MAX_RETRIES = 3
```

**糟糕示例：魔法数字：**

```python  theme={null}
TIMEOUT = 47  # Why 47?
RETRIES = 5   # Why 5?
```

### 提供实用脚本

即使代理能够编写脚本，预先制作好的脚本也有优势：

**实用脚本的优点：**

* 比生成的代码更可靠
* 节省 token（无需把代码放入上下文）
* 节省时间（无需生成代码）
* 确保不同使用场景的一致性

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=4bbc45f2c2e0bee9f2f0d5da669bad00" alt="将可执行脚本与指令文件放在一起" data-og-width="2048" width="2048" data-og-height="1154" height="1154" data-path="images/agent-skills-executable-scripts.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=9a04e6535a8467bfeea492e517de389f 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=e49333ad90141af17c0d7651cca7216b 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=954265a5df52223d6572b6214168c428 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=2ff7a2d8f2a83ee8af132b29f10150fd 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=48ab96245e04077f4d15e9170e081cfb 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0301a6c8b3ee879497cc5b5483177c90 2500w" />

上图展示了可执行脚本如何与指令文件配合使用。指令文件（forms.md）引用脚本，代理可以执行脚本而无需加载其内容到上下文中。

**重要区分：**明确说明代理应该：

* **执行脚本**（最常见）：“运行 `analyze_form.py` 提取字段”
* **将脚本作为参考读取**（适用于复杂逻辑）：“参见 `analyze_form.py` 了解字段提取算法”

对于大多数实用脚本，推荐执行，因为它更可靠、更高效。更多细节参见下面的[运行时环境](#runtime-environment)章节。

**示例：**

````markdown  theme={null}
## Utility scripts

**analyze_form.py**: Extract all form fields from PDF

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

Output format:
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**: Check for overlapping bounding boxes

```bash
python scripts/validate_boxes.py fields.json
# Returns: "OK" or lists conflicts
```

**fill_form.py**: Apply field values to PDF

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
````

### 使用视觉分析

当输入可以渲染为图像时，让代理分析它们：

````markdown  theme={null}
## Form layout analysis

1. Convert PDF to images:
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. Analyze each page image to identify form fields
3. The agent can see field locations and types visually
````

<Note>
  在这个示例中，你需要编写 `pdf_to_images.py` 脚本。
</Note>

代理的视觉能力有助于理解布局和结构。

### 创建可验证的中间输出

代理执行复杂的开放式任务时可能出错。“计划—验证—执行”模式可以让代理先以结构化格式创建计划，再用脚本验证计划，从而尽早捕获错误。

**示例：**想象让代理根据一个电子表格更新 PDF 中的 50 个表单字段。没有验证，它可能引用不存在的字段、创建冲突的值、遗漏必填字段，或错误应用更新。

**解决方案：**使用前面展示的 PDF 表单填写工作流，但增加一个在应用更改前进行验证的中间 `changes.json` 文件。工作流变为：分析 → **创建计划文件** → **验证计划** → 执行 → 验证。

**这种模式有效的原因：**

* **尽早捕获错误：**在应用更改前发现问题
* **机器可验证：**脚本提供客观验证
* **可逆规划：**代理可以迭代计划而不触碰原始文件
* **清晰调试：**错误消息指向具体问题

**适用时机：**批量操作、破坏性更改、复杂验证规则、高风险操作。

**实现提示：**让验证脚本输出具体且详细的错误消息，例如“找不到字段 `signature_date`。可用字段：`customer_name`、`order_total`、`signature_date_signed`”，帮助代理修复问题。

### 软件包依赖

技能运行在具有平台特定限制的代码执行环境中：

* **claude.ai：**可以从 npm 和 PyPI 安装软件包，并从 GitHub 拉取仓库
* **Anthropic API：**无法访问网络，也无法在运行时安装软件包

在 SKILL.md 中列出所需软件包，并在[代码执行工具文档](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool)中确认它们可用。

### 运行时环境

技能运行在具备文件系统访问、bash 命令和代码执行能力的代码执行环境中。关于此架构的概念解释，参见概览中的 [Skills architecture](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#the-skills-architecture)。

**这对编写技能的影响：**

**代理如何访问技能：**

1. **预加载元数据：**启动时，所有技能 YAML frontmatter 中的名称和描述会加载到系统提示词
2. **按需读取文件：**代理使用文件读取工具访问需要的 SKILL.md 和其他文件
3. **高效执行脚本：**脚本可以通过 bash 执行，而无需加载完整内容。只有脚本输出会消耗 token
4. **大型文件没有上下文惩罚：**参考文件、数据或文档在实际访问前不会消耗上下文 token

* **文件路径很重要：**代理像浏览文件系统一样浏览技能目录。使用正斜杠（`reference/guide.md`），不要使用反斜杠
* **使用描述性文件名：**名称应体现内容：使用 `form_validation_rules.md`，不要使用 `doc2.md`
* **为发现而组织：**按领域或功能组织目录
  * 好：`reference/finance.md`、`reference/sales.md`
  * 坏：`docs/file1.md`、`docs/file2.md`
* **打包完整资源：**包含完整 API 文档、大量示例、大型数据集；只有访问时才会产生上下文成本
* **优先使用脚本执行确定性操作：**编写 `validate_form.py`，而不是要求代理生成验证代码
* **明确执行意图：**
  * “运行 `analyze_form.py` 提取字段”（执行）
  * “参见 `analyze_form.py` 了解提取算法”（作为参考读取）
* **测试文件访问模式：**使用真实请求验证代理能否浏览目录结构

**示例：**

```
bigquery-skill/
├── SKILL.md (overview, points to reference files)
└── reference/
    ├── finance.md (revenue metrics)
    ├── sales.md (opportunities, pipeline)
    └── product.md (usage analytics)
```

当用户询问收入时，代理读取 SKILL.md，看到对 `reference/finance.md` 的引用，然后调用 bash 读取该文件。sales.md 和 product.md 留在文件系统中，在需要访问前不会消耗上下文 token。这种基于文件系统的模型支持渐进式披露，让代理能够浏览并按需加载精确内容。

关于完整技术架构，参见技能概览中的 [How Skills work](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)。

### MCP 工具引用

如果技能使用 MCP 工具，始终使用完全限定的工具名称，以避免“找不到工具”错误。

**格式：**`ServerName:tool_name`

**示例：**

```markdown  theme={null}
Use the BigQuery:bigquery_schema tool to retrieve table schemas.
Use the GitHub:create_issue tool to create issues.
```

其中 `BigQuery` 和 `GitHub` 是 MCP 服务器名称，`bigquery_schema` 和 `create_issue` 是服务器中的工具名称。

如果没有服务器前缀，尤其是在有多个 MCP 服务器可用时，代理可能无法定位工具。

### 避免假设工具已安装

不要假设软件包可用：

````markdown  theme={null}
**Bad example: Assumes installation**:
"Use the pdf library to process the file."

**Good example: Explicit about dependencies**:
"Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
````

## 技术说明

### YAML frontmatter 要求

SKILL.md 的 frontmatter 要求 `name`（最多 64 个字符）和 `description`（最多 1024 个字符）字段。关于完整结构细节，参见 [Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#skill-structure)。

### Token 预算

为获得最佳性能，保持 SKILL.md 正文少于 500 行。如果内容超过这一长度，使用前面描述的渐进式披露模式将内容拆分到单独文件。

## 有效技能清单

分享技能前，进行以下验证：

### 核心质量

* [ ] 描述具体且包含关键术语
* [ ] 描述同时包含技能做什么以及何时使用
* [ ] SKILL.md 正文少于 500 行
* [ ] 需要时将附加细节放入单独文件
* [ ] 没有时间敏感信息（或放在“旧模式”章节）
* [ ] 全文术语一致
* [ ] 示例具体而非抽象
* [ ] 文件引用只有一层深度
* [ ] 适当使用渐进式披露
* [ ] 工作流步骤清晰

### 代码和脚本

* [ ] 脚本解决问题，而不是把问题推给代理
* [ ] 错误处理明确且有帮助
* [ ] 没有“巫术常量”（所有值都有理由）
* [ ] 指令中列出所需软件包，并确认它们可用
* [ ] 脚本有清晰文档
* [ ] 没有 Windows 风格路径（全部使用正斜杠）
* [ ] 关键操作有验证/核验步骤
* [ ] 质量关键任务包含反馈循环

### 测试

* [ ] 至少创建三个评估
* [ ] 使用 Haiku、Sonnet 和 Opus 进行了测试
* [ ] 使用真实场景进行了测试
* [ ] 纳入了团队反馈（如适用）

## 后续步骤

<CardGroup cols={2}>
  <Card title="Get started with Agent Skills" icon="rocket" href="https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart">
    Create your first Skill
  </Card>

  <Card title="Use Skills in Claude Code" icon="terminal" href="https://code.claude.com/docs/en/skills">
    Create and manage Skills in Claude Code
  </Card>

  <Card title="Use Skills with the API" icon="code" href="https://platform.claude.com/docs/en/build-with-claude/skills-guide">
    Upload and use Skills programmatically
  </Card>
</CardGroup>
