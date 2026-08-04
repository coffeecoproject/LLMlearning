---
type: mechanism-note
module: 3
learning_layer: persistent-high-reliability
status: complete
audience: non-specialist
parent: "[[00-从模糊需求到正式Goal概览|从模糊需求到正式 Goal 概览]]"
tags: [goal, scope, context, execution, acceptance]
---

# Goal 如何约束后续执行

> [!summary]
> 正式 Goal 不是创建后就被遗忘的说明文字；它应持续参与上下文编译、计划检查、权限判断、变更范围和最终验收。

> [!note] 整篇层级：持续与高可靠结构（选读）
> 基础 Agent 只需让当前请求持续影响后续模型调用；正式 Goal 向计划、权限、版本和验收传播属于结构化任务控制。

## Goal 向下游传播

```text
Goal revision 1
├── Discovery：必须回答哪些未知项
├── Plan：哪些任务与目标相关
├── Context：本阶段必须带入哪些条件
├── Permission：哪些动作在 Scope 内
├── Candidate：实际变更是否符合允许范围
├── Verification：每条成功条件怎样检查
└── Acceptance：证据是否足以宣布完成
```

## 对 Discovery 的约束

Discovery 不应“把整个仓库都研究一遍”，而应围绕 Goal 查找：

- 相关业务路径；
- 真实影响范围；
- 关键未知项；
- 需要用户决定的问题。

## 对 Plan 和 Tool 的约束

Plan 中的每项工作应能追溯到 Goal、事实或验证义务。Tool Call 还要检查当前动作是否超出 Scope。

例如 Goal 明确“不处理历史订单”，Agent 就不能以“顺便清理数据”为理由运行批量退款工具。

## 对 Context 的约束

每一轮模型不需要收到全部 Goal 历史，但必须收到当前阶段不可缺少的：

- Goal 身份和 revision；
- 当前相关成功条件；
- Scope 和 Non-goals；
- 已确认决定和阻塞项。

这样即使使用新线程，模型仍能在当前权威边界内工作。

## 对验证和完成的约束

模型的 Completion Request 必须重新映射到 Goal：

```text
每条 required criterion
→ 是否存在当前有效的 Verification Obligation
→ 是否获得绑定正确版本的 Evidence
→ 是否存在未解决阻塞或范围遗漏
```

只有这些条件满足，Acceptance 才能批准结束。

## Goal 改变时会发生什么

如果用户后来要求“历史重复扣款也要退款”，这不是普通 Plan 调整，而是 Goal Scope 改变。

系统应创建新 revision，并检查：

- 旧 Plan 是否仍适用；
- 当前 Candidate 是否覆盖新范围；
- 旧 Context 是否过期；
- 已有 Evidence 是否仍能证明新 Goal；
- 是否需要新的权限和用户批准。

## 最终因果链

```text
高质量 Goal
→ 更准确的探索和计划边界
→ 更小的越权和遗漏风险
→ 更明确的验证义务
→ 更可信的完成判断
```

但 Goal 只是必要条件，不是充分条件；后续仍需要可靠状态、工具执行和验收系统。
