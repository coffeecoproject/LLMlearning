---
type: reference-note
module: 3
learning_layer: cross-layer
status: complete
audience: non-specialist
parent: "[[00-架构模式与搭建路线概览|架构模式与搭建路线概览]]"
tags: [workflow, agent-loop, runtime, loop-engineering]
---

# Workflow、Agent Loop 与 Loop Engineering

> [!summary]
> Workflow 是预定义控制路径，Agent Loop 是运行中的模型—行动—观察循环，Agent Runtime 是承载循环的程序；Loop Engineering 则是设计和改进整套循环系统的工程实践。

> [!info] 两层边界
> **基础 Agent 结构**只需要 Agent Loop 和承载它的 Runtime；正式 Workflow、状态机和系统化 Loop Engineering 属于复杂任务或持续与高可靠结构，并非每个 Agent 都必须拥有。

## 四个概念不是同义词

| 概念 | 核心问题 |
|---|---|
| Workflow | 系统预先规定哪些阶段和转换合法 |
| Agent Loop | 模型怎样根据 Observation 动态决定下一步 |
| Agent Runtime / Harness | 谁构建上下文、调用模型、执行工具并维持循环 |
| Loop Engineering | 怎样把目标、状态、上下文、工具、验证和停止条件设计成可靠闭环 |

## Workflow 什么时候成立

```text
Goal Confirmed
→ Discovery
→ Implementation
→ Verification
→ Accepted
```

如果这些阶段和进入条件由程序预先规定，它就是 Workflow 或状态机。仅有流程定义不能自动运行，还需要执行器、状态存储、模型和工具；但这不意味着它不能叫 Workflow。

## Agent Loop 什么时候成立

```text
构建上下文
→ 调用模型
→ 模型请求工具
→ Runtime 执行工具
→ 结果成为 Observation
→ 再次调用模型
→ 直到最终输出或停止
```

这里的下一步主要由模型根据环境反馈动态决定，因此称为 Agent Loop。

## 两者怎样组合

```text
外层 Workflow
├── DISCOVERY
│   └── Codex Agent Loop：搜索和理解项目
├── IMPLEMENT
│   └── Codex Agent Loop：修改并读取反馈
└── VERIFY
    ├── 确定性测试
    └── 可选独立 Reviewer
```

外层提供阶段、权限和完成边界；内层利用模型能力处理无法提前写死的具体步骤。

## Loop Engineering 在设计什么

Loop Engineering 不只是写一个 `while` 循环，而是回答：

- 目标怎样形成和修订；
- 每次 Model Call 应该看到什么；
- 哪些工具可以执行；
- Observation 怎样进入下一次决策；
- 哪些状态需要持久化；
- 怎样限制 Token、时间、费用和重试；
- 什么证据允许继续、修复、停止或升级给人；
- 模型升级后哪些控制假设可以删除，哪些仍然必要。

## 术语成熟度

`Agent Loop` 已经是 OpenAI 等 Agent 文档直接使用的运行术语。`Workflow` 与 `Agent` 的预定义路径／动态决策差异也有较稳定的行业用法。`Loop Engineering` 则是 2026 年快速流行的工程概括，适合表示“设计 Agent 外部反馈闭环”的实践，但尚不应当成所有架构的唯一标准名称。

## 来源与核对时间

核对日期：2026-07-30。

- [OpenAI：Unrolling the Codex agent loop](https://openai.com/index/unrolling-the-codex-agent-loop/)；
- [OpenAI Agents SDK：The agent loop](https://openai.github.io/openai-agents-python/running_agents/#the-agent-loop)；
- [Anthropic：Workflows 与 Agents 的架构区别](https://www.anthropic.com/engineering/building-effective-agents)；
- [IBM：Loop Engineering 是一项新兴 Agent 工程实践](https://www.ibm.com/think/topics/loop-engineering)。

## 停止点

如果你能说明“Workflow 规定外层阶段，Agent Runtime 在局部阶段中运行 Agent Loop，而 Loop Engineering 负责设计这套反馈和控制系统”，就已经达到本概念的框架理解，不必继续追逐更多新术语。
