# 技能设计中的说服原则

## 概述

LLM 对说服原则的反应与人类相似。理解这些心理学原理有助于你设计更有效的技能——不是为了操纵，而是为了确保关键实践即使在压力下也能被遵循。

**研究基础：**Meincke 等人（2025）以 N=28,000 次 AI 对话测试了 7 条说服原则。说服技巧使合规率从 33% 提高到 72% 以上（p < .001）。

## 七条原则

### 1. 权威
**含义：**对专业知识、资历或官方来源的服从。

**在技能中的作用方式：**
- 祈使式语言：“YOU MUST”“Never”“Always”
- 不可协商的表述：“No exceptions”
- 消除决策疲劳和理由化倾向

**适用场景：**
- 强制纪律的技能（TDD、验证要求）
- 安全关键实践
- 已建立的最佳实践

**示例：**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. 承诺
**含义：**与先前的行动、陈述或公开声明保持一致。

**在技能中的作用方式：**
- 要求宣布：“宣布技能的使用情况”
- 迫使明确选择：“选择 A、B 或 C”
- 使用跟踪：待办事项作为清单

**适用场景：**
- 确保技能确实被遵循
- 多步骤流程
- 问责机制

**示例：**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. 稀缺
**含义：**来自时间限制或有限可用性的紧迫感。

**在技能中的作用方式：**
- 有时间约束的要求：“在继续之前”
- 顺序依赖：“在 X 之后立即执行 Y”
- 防止拖延

**适用场景：**
- 需要立即验证
- 有时间限制的工作流
- 防止出现“稍后再做”

**示例：**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. 社会认同
**含义：**遵从他人的做法，或遵从被认为正常的做法。

**在技能中的作用方式：**
- 普遍化模式：“每次”“始终”
- 失败模式：“没有 Y 的 X = 失败”
- 建立规范

**适用场景：**
- 记录通用实践
- 警告常见失败
- 强化标准

**示例：**
```markdown
✅ Checklists without todo tracking = steps get skipped. Every time.
❌ Some people find a todo list helpful for checklists.
```

### 5. 统一
**含义：**共同身份，“我们感”，即群体内的归属感。

**在技能中的作用方式：**
- 协作式语言：“我们的代码库”“我们是同事”
- 共同目标：“我们都希望质量更高”

**适用场景：**
- 协作工作流
- 建立团队文化
- 非层级式实践

**示例：**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. 互惠
**含义：**有义务回报所获得的利益。

**作用方式：**
- 谨慎使用——可能让人感觉受到操纵
- 技能中很少需要

**应避免的场景：**
- 几乎总是应避免（其他原则更有效）

### 7. 喜爱
**含义：**偏好与自己喜欢的人合作。

**作用方式：**
- **不要将其用于合规**
- 与诚实反馈文化相冲突
- 会制造迎合

**应避免的场景：**
- 始终避免将其用于纪律约束

## 按技能类型组合原则

| 技能类型 | 使用 | 避免 |
|------------|-----|-------|
| 强制纪律 | 权威 + 承诺 + 社会认同 | 喜爱、互惠 |
| 指导/技术 | 适度权威 + 统一 | 过度权威 |
| 协作型 | 统一 + 承诺 | 权威、喜爱 |
| 参考型 | 只需清晰 | 所有说服原则 |

## 为什么有效：心理学

**明线规则减少理由化：**
- “YOU MUST”消除决策疲劳
- 绝对化语言消除“这算例外吗？”的问题
- 明确的反理由化表述可以堵住特定漏洞

**执行意图会创造自动行为：**
- 清晰的触发条件 + 必须执行的动作 = 自动执行
- “当 X 时，执行 Y”比“通常执行 Y”更有效
- 降低合规的认知负担

**LLM 是类人系统：**
- 它们在包含这些模式的人类文本上训练
- 训练数据中，权威语言常常先于合规出现
- 承诺序列（声明 → 行动）经常被建模
- 社会认同模式（所有人都做 X）会建立规范

## 伦理使用

**正当使用：**
- 确保关键实践得到遵循
- 创建有效文档
- 防止可预见的失败

**不正当使用：**
- 为个人利益进行操纵
- 制造虚假紧迫感
- 以负罪感迫使合规

**检验标准：**如果用户完全理解这种技巧，它是否仍然服务于用户的真实利益？

## 研究引用

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- 说服的七条原则
- 影响力研究的实证基础

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 以 N=28,000 次 LLM 对话测试了 7 条原则
- 使用说服技巧后，合规率从 33% 提高到 72%
- 权威、承诺、稀缺最有效
- 验证了 LLM 行为的类人模型

## 快速参考

设计技能时，问自己：

1. **它属于哪种类型？**（纪律、指导还是参考）
2. **我想改变哪种行为？**
3. **哪些原则适用？**（纪律技能通常使用权威 + 承诺）
4. **我是否组合了太多原则？**（不要七条全用）
5. **这符合伦理吗？**（是否服务于用户的真实利益？）
