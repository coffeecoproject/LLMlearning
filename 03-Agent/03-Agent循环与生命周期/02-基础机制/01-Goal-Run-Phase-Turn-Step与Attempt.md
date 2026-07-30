---
type: mechanism-note
module: 3
status: complete
audience: non-specialist
parent: "[[00-运行对象与状态机概览|运行对象与状态机概览]]"
tags: [goal, run, phase, turn, step, attempt]
---

# Goal、Run、Phase、Turn、Step 与 Attempt

> [!summary]
> 这些对象把一个长期任务拆成不同粒度的可追踪单位，使系统能够定位状态、失败和恢复范围。

## 一层层看

```text
Goal：修复重复扣款
└── Run：本次受控执行
    ├── Phase：DISCOVERY
    │   └── Attempt 1
    │       ├── Turn 1
    │       ├── Step：执行搜索工具
    │       └── Turn 2
    ├── Phase：IMPLEMENT
    │   └── Attempt 1
    └── Phase：VERIFY
        └── Attempt 1
```

这是一种说明性层级，不同框架可能命名不同。

## 每个对象解决什么

| 对象 | 解决的问题 |
|---|---|
| Goal | 最终为什么执行、怎样算完成 |
| Run | 哪一次实际运行正在处理 Goal |
| Phase | 当前阶段目的、权限和退出条件是什么 |
| Attempt | 这是对当前阶段或任务的第几次尝试 |
| Turn | 哪一次模型请求和响应 |
| Step | Runtime 执行或记录的哪个动作 |

## Thread 在哪里

Thread 通常是模型或 Agent 产品中的对话会话。它可以承载多个 Turn，但不必成为权威 Run 身份。

```text
一个 Run
→ 可以续接同一个 Thread
→ 也可以在不同 Phase 使用新 Thread

一个 Thread 丢失
→ 不应导致 Run 状态丢失
```

## Attempt 为什么重要

假设第一次实现失败：

```text
IMPLEMENT Attempt 1
→ 修改 A
→ 测试失败

IMPLEMENT Attempt 2
→ 根据失败证据重新修改
```

如果不区分 Attempt，旧结果、错误和新结果容易混在一起，恢复时也难以判断哪些动作可以重做。

## 不要过度建模

最小 Agent 不一定一开始就需要全部对象。可以从：

```text
Goal + Run + Turn + Tool Call
```

开始。当任务需要阶段权限、重试和恢复时，再引入 Phase、Step 和 Attempt。但概念上必须知道它们解决的是不同粒度问题。
