---
name: requesting-code-review
description: 在完成任务、实现重大功能或合并前使用，以验证工作符合要求
---

# 请求代码审查

派遣代码审查子代理，在问题扩散之前发现问题。审查者会获得为评估精心编写的准确上下文——绝不使用你会话的历史。

**核心原则：** 尽早审查，经常审查。

## 何时请求审查

**强制：**
- 在子代理驱动开发中的每个任务之后
- 完成重大功能之后
- 合并到 main 之前

**可选但很有价值：**
- 卡住时（获得新的视角）
- 重构之前（进行基线检查）
- 修复复杂 bug 之后

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派遣代码审查子代理：**

派遣一个 `general-purpose` 子代理，填写 [code-reviewer.md](code-reviewer.md) 中的模板。

**占位符：**
- `{DESCRIPTION}` - 你构建内容的简短摘要
- `{PLAN_OR_REQUIREMENTS}` - 它应该做什么
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交

**3. 处理反馈：**
- 立即修复 Critical 问题
- 在继续之前修复 Important 问题
- 记录 Minor 问题，留待之后处理
- 如果审查者有误，说明理由并提出异议

## 示例

```
[Just completed Task 2: Add verification function]

You: Let me request code review before proceeding.

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[Dispatch code reviewer subagent]
  DESCRIPTION: Added verifyIndex() and repairIndex() with 4 issue types
  PLAN_OR_REQUIREMENTS: Task 2 from docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661

[Subagent returns]:
  Strengths: Clean architecture, real tests
  Issues:
    Important: Missing progress indicators
    Minor: Magic number (100) for reporting interval
  Assessment: Ready to proceed

You: [Fix progress indicators]
[Continue to Task 3]
```

## 常见合理化借口

| 借口 | 现实 |
|--------|---------|
| "I'll just review the diff myself instead of dispatching a reviewer" | 你是协调者——在内联审查 diff 时会消耗本应用于继续推进工作的上下文窗口。派遣审查子代理：diff 和评估都在它的上下文中，只有发现结果会返回给你。 |
| "The reviewer needs my whole session history to understand the change" | 给它精心编写的准确上下文，绝不要给你的会话历史。这样审查者关注的是工作产物，而不是你的思考过程。 |

## 红旗信号

**绝不要：**
- 因为“很简单”而跳过审查
- 忽略 Critical 问题
- 带着未修复的 Important 问题继续
- 反驳有效的技术反馈

**如果审查者有误：**
- 用技术推理提出异议
- 展示证明其可工作的代码/测试
- 请求澄清

参见模板：[code-reviewer.md](code-reviewer.md)
