---
name: verification-before-completion
description: 在即将声称工作已完成、已修复或已通过，并且准备提交或创建 PR 之前使用——要求运行验证命令并确认输出后才能做出任何成功声明；始终以证据先于断言
---

# 完成前验证

## 概述

**核心原则：**始终以证据支持声明。

违反这条规则的字面要求，就是违反这条规则的精神。

## 铁律

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

如果你没有在本条消息中运行验证命令，就不能声称它通过了。

## 闸门函数

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## 常见失败

| 声明 | 需要 | 不足以证明 |
|------|------|------------|
| 测试通过 | 测试命令输出：0 个失败 | 之前的运行结果、“应该通过” |
| Linter 无问题 | Linter 输出：0 个错误 | 部分检查、推断 |
| 构建成功 | 构建命令：退出码 0 | Linter 通过、日志看起来正常 |
| Bug 已修复 | 原始症状测试：通过 | 代码已修改、假定已修复 |
| 回归测试有效 | 已验证 Red-Green 循环 | 测试只通过一次 |
| 代理已完成 | VCS diff 显示变更 | 代理报告“成功” |
| 满足要求 | 逐行检查清单 | 测试通过 |

## 危险信号——停止

- 使用“应该”“可能”“看起来像”等词
- 在验证前表达满意（“太好了！”、“完美！”、“完成了！”等）
- 即将做出暗示成功的任何表述
- 未验证就提交、创建 PR
- 依赖部分验证
- 认为“就这一次”
- 感到疲惫、想结束工作
- **任何暗示工作成功而没有运行验证的措辞**

## 防止找借口

| 借口 | 事实 |
|------|------|
| “现在应该能工作了” | 运行验证。 |
| “我很有把握” | 把握 ≠ 证据。 |
| “就这一次” | 没有例外。 |
| “Linter 通过了” | Linter ≠ 编译器。 |
| “代理说成功了” | 独立验证。 |
| “我很累” | 疲惫不是借口。 |
| “部分检查就够了” | 部分检查无法证明整体。 |
| “换一种说法就不适用这条规则了” | 重视规则精神，而不是钻字面空子。 |

## 关键模式

**测试：**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**回归测试（TDD Red-Green）：**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**构建：**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**要求：**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**代理委派：**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## 何时应用

**始终在以下事项之前：**

- 任何形式的成功/完成声明
- 任何满意的表达
- 任何关于工作状态的正面表述
- 提交、创建 PR、完成任务
- 进入下一项任务
- 委派代理

**规则适用于：**

- 精确措辞
- 改写说法
- 任何暗示
- **任何暗示正确性/完成性的沟通**
