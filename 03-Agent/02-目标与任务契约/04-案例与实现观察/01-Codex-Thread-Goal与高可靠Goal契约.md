---
type: implementation-observation
module: 3
learning_layer: persistent-high-reliability
status: complete
audience: non-specialist
parent: "[[00-目标与任务契约概览|目标与任务契约概览]]"
tags: [agent, codex, thread-goal, goal-contract, implementation-boundary]
verified: 2026-07-30
---

# Codex Thread Goal 与高可靠 Goal 契约

> [!summary]
> Codex CLI 已经拥有线程级持久 Goal，但它主要保存一个持续 Objective 和运行状态；高可靠 Goal Contract 则进一步把目标边界、成功条件、版本和验收关系变成独立对象。

## 当前 Codex 的真实主线

```text
用户使用 /goal <objective>
或明确要求模型创建 Goal
        ↓
Codex 持久化 Thread Goal
        ↓
同一个线程继续
理解 → 调查 → 规划 → 工具执行 → 检查结果
        ↓
模型调用 update_goal
标记 complete 或 blocked
```

官方文档说明 `/goal` 可以设置、查看、编辑、暂停、恢复或清除当前任务目标，并把它附着在当前任务上持续追踪：[Codex `/goal` 文档](https://learn.chatgpt.com/docs/developer-commands#set-or-view-a-task-goal-with-goal)。

## Thread Goal 保存什么

当前公开协议包含：

```text
ThreadGoal
├── objective
├── status
├── tokenBudget
├── tokensUsed
├── timeUsedSeconds
├── createdAt
└── updatedAt
```

状态数据库内部还保存 `thread_id` 和 `goal_id`。来源：

- [ThreadGoal 协议](https://github.com/openai/codex/blob/ff352fab6209dc0f9d13fc0036ed3f9404682b2c/codex-rs/app-server-protocol/schema/typescript/v2/ThreadGoal.ts)；
- [ThreadGoal 状态模型](https://github.com/openai/codex/blob/ff352fab6209dc0f9d13fc0036ed3f9404682b2c/codex-rs/state/src/model/thread_goal.rs)；
- [Thread Goal SQLite 结构](https://github.com/openai/codex/blob/ff352fab6209dc0f9d13fc0036ed3f9404682b2c/codex-rs/state/goals_migrations/0001_thread_goals.sql)。

这证明它不是仅存在于聊天文字中的临时目标。它能够跨 Turn 保存，并在恢复同一持久线程后继续使用。

## 源码中的 GoalDraft 不是需求分析草稿

Codex TUI 确实有一个名为 `GoalDraft` 的结构，但它保存的是用户正在输入的 Objective、粘贴文本和图片，随后被整理成最终 Objective：[goal_files.rs](https://github.com/openai/codex/blob/ff352fab6209dc0f9d13fc0036ed3f9404682b2c/codex-rs/tui/src/goal_files.rs)。

```text
Codex TUI GoalDraft
= 设置 Goal 前的界面输入草稿

本专题所说的 Goal Draft
= LLM 对需求的候选理解、成功条件、范围、假设和澄清问题
```

两者名称相同，但系统职责不同。

## Codex 怎样持续完成 Goal

Goal Runtime 会把持久 Objective、已用预算和剩余预算整理成内部继续信息，在线程空闲时启动后续 Turn。它要求模型：

- 保持完整 Objective，不把目标缩小成当前容易完成的部分；
- 根据真实工作区和外部状态继续工作；
- 从 Objective 和引用资料中推导具体要求；
- 逐项检查文件、命令、测试和其他证据；
- 证据不足时继续工作；
- 完成后调用 `update_goal(complete)`。

来源：[Goal continuation 模板](https://github.com/openai/codex/blob/ff352fab6209dc0f9d13fc0036ed3f9404682b2c/codex-rs/ext/goal/templates/goals/continuation.md)和[Goal Runtime](https://github.com/openai/codex/blob/ff352fab6209dc0f9d13fc0036ed3f9404682b2c/codex-rs/ext/goal/src/runtime.rs)。

## 与高可靠 Goal Contract 对比

| 问题 | Codex Thread Goal | 高可靠 Goal Contract |
|---|---|---|
| 主要范围 | 当前持久线程 | 项目或业务任务运行 |
| 目标内容 | 主要是一个 Objective | Objective、Criteria、Scope、Non-goals 等 |
| Goal Intake | 没有强制独立阶段 | 可以先生成 Draft 并请求用户确认 |
| 运行方式 | 同一 Agent Loop 灵活调查和执行 | 可以按 Phase 隔离权限和上下文 |
| 完成检查 | 模型读取真实证据后自我审计 | 独立系统把 Criteria、Candidate 和 Evidence 对应 |
| 完成权威 | 模型调用 `update_goal`，Runtime 持久化 | Acceptance Engine 或其他外部权威决定 |
| 版本变化 | 可以编辑或替换 Objective | Goal revision 触发计划、产物和证据失效检查 |

## 应怎样评价 Codex

Codex 不是“没有 Goal 的简单 Agent”。它已经属于持续 Agent：目标能够持久化、恢复、计量并驱动自动继续。

但也不能把它直接等同于本专题的完整任务契约系统。Codex 当前更依赖同一个模型根据 Objective 推导要求、执行任务并完成审计；高可靠架构则进一步把这些推导结果和完成权威外部化。

## 事实、推断与未知

### 公开源码可以确认

- Thread Goal 会持久化；
- Goal 与 Thread 绑定；
- Goal 保存 Objective、状态和预算使用；
- Runtime 可以在 Goal 活跃时继续启动 Turn；
- 模型可以通过 Goal 工具创建目标或标记完成，但创建只允许响应用户或系统的明确要求，见 [Goal 工具定义](https://github.com/openai/codex/blob/ff352fab6209dc0f9d13fc0036ed3f9404682b2c/codex-rs/ext/goal/src/spec.rs)。

### 根据源码作出的架构判断

- Codex 属于“持久 Thread Goal + 一体化 Agent Loop”；
- 它的完成审计比普通自然语言自报完成更严格，但不等于独立 Acceptance Engine。

### 当前公开源码没有证明

- 存在强制独立运行的 Goal Intake Agent；
- Success Criteria、Scope 和 Evidence 已作为 Thread Goal 的独立权威字段；

## 一句话复习

```text
Codex Thread Goal
= 让同一个编程 Agent 长时间记住并持续追踪一个目标

高可靠 Goal Contract
= 让目标怎样形成、怎样修改、怎样验证和谁能宣布完成都成为受控系统状态
```
