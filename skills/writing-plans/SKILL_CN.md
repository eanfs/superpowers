---
name: writing-plans
description: 在已有规范或需求、且任务包含多个步骤并准备接触代码之前使用
---

# 编写计划

## 概述

编写全面的实现计划，假定工程师对我们的代码库完全不了解，而且品味值得怀疑。记录他们需要知道的一切：每项任务要接触哪些文件、代码、需要测试的内容、可能需要查阅的文档，以及如何测试。把完整计划拆成细小任务。DRY。YAGNI。TDD。频繁提交。

假定他们是熟练的开发者，但几乎不了解我们的工具集或问题领域。假定他们对良好的测试设计也不太熟悉。

**开始时宣布：**“I'm using the writing-plans skill to create the implementation plan.”

**上下文：**如果在隔离的 worktree 中工作，应在执行时通过 `superpowers:using-git-worktrees` 技能创建该 worktree。

**保存计划到：**`docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- （用户对计划位置的偏好覆盖此默认位置。）

## 范围检查

如果规范涉及多个相互独立的子系统，就应该在头脑风暴阶段拆分成多个子项目规范。如果尚未拆分，请建议拆成多个计划——每个子系统一个。每个计划都应该独立地产出可运行、可测试的软件。

## 文件结构

在定义任务之前，先梳理将要创建或修改的文件，以及每个文件负责什么。这一步会锁定拆分决策。

- 设计具有清晰边界和明确接口的单元。每个文件只有一个清晰职责。
- 你对能一次放在上下文中的代码推理效果最好；文件聚焦时，编辑也更可靠。优先选择小而聚焦的文件，而不是让大文件承担过多职责。
- 一起变更的文件应放在一起。按职责拆分，而不是按技术层拆分。
- 在现有代码库中遵循既有模式。如果代码库使用大文件，不要单方面重构——但如果要修改的文件已经难以维护，在计划中纳入拆分是合理的。

这个结构会为任务分解提供依据。每项任务都应产出独立且有意义的变更。

## 任务大小

任务是一个拥有自身测试周期、值得经过一次新鲜审查闸门的最小单元。划分任务边界时：把设置、配置、脚手架和文档步骤并入需要这些内容的交付任务；只有在审查者可以拒绝一项任务而批准相邻任务时才拆分。每项任务都以一个可独立测试的交付物结束。

## 细小任务粒度

**每一步都是一个动作（2–5 分钟）：**
- “编写失败测试”——一步
- “运行它以确认失败”——一步
- “编写让测试通过的最小代码”——一步
- “运行测试并确认通过”——一步
- “提交”——一步

## 计划文档头部

**每个计划都必须以此头部开始：**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Spec:** [path to the spec/design doc this plan implements — the plan
argues from the spec, so the spec travels with it; executors read both]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

---
```

## 任务结构

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 不得使用占位符

每一步都必须包含工程师所需的实际内容。以下都是**计划失败**——绝不能写：
- “TBD”“TODO”“稍后实现”“补充细节”
- “添加适当的错误处理”/“添加验证”/“处理边界情况”
- “为上述内容编写测试”（没有实际测试代码）
- “与任务 N 类似”（重复代码——工程师可能按不同顺序阅读任务）
- 只描述要做什么而不展示如何做的步骤（代码步骤必须包含代码块）
- 引用任何任务都没有定义的类型、函数或方法

## 自我审查

完成整份计划后，用全新的视角检查规范和计划。

**1. 规范覆盖：**浏览规范的每个章节/要求。能否指出负责实现它的任务？列出所有缺口。

**2. 占位符扫描：**搜索计划中的危险信号——上面“不得使用占位符”部分的所有模式。修复它们。

**3. 类型一致性：**后续任务中使用的类型、方法签名和属性名称，是否与前面任务定义的内容一致？任务 3 中名为 `clearLayers()` 的函数，在任务 7 中却叫 `clearFullLayers()`，这就是一个 bug。

如果发现问题，就在原处修复。不需要重新审查——直接修复并继续。如果发现某项规范要求没有对应任务，就添加任务。

## 执行交接

保存计划后，提供执行选项：

**“计划已完成并保存到 `docs/superpowers/plans/<filename>.md`。有两种执行方式：**

**1. 子代理驱动（推荐）**——为每项任务派发一个全新的子代理，并在任务之间进行审查，快速迭代

**2. 行内执行**——在本会话中使用 executing-plans 执行任务，分批执行并在检查点停下供审查

**请选择一种方式？”**

**如果选择子代理驱动：**
- **必需子技能：**使用 superpowers:subagent-driven-development
- 每项任务使用全新的子代理 + 两阶段审查

**如果选择行内执行：**
- **必需子技能：**使用 superpowers:executing-plans
- 批量执行并在检查点停下供审查
