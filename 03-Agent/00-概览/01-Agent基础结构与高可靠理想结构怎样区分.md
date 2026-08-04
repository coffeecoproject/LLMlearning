---
type: framework
module: 3
learning_layer: cross-layer
status: complete
audience: non-specialist
parent: "[[00-Agent模块大纲|Agent 模块大纲]]"
previous: "[[00-Agent模块大纲|Agent 模块大纲]]"
next: "[[00-Agent框架速览概览|Agent 框架速览概览]]"
tags: [agent, architecture, minimal-agent, persistent-agent, high-reliability]
---

# Agent 基础结构与高可靠理想结构怎样区分

> [!summary]
> Agent 的基础结构只要求模型能够在外部程序控制下根据环境反馈选择下一步；持久状态、正式任务契约、Context Manifest、独立 Evidence 与 Acceptance 都是按任务长度和风险逐步增加的工程能力，不是 Agent 成立的定义条件。

## 为什么必须分层

如果直接从一套高可靠系统开始学习，容易得到错误结论：

~~~text
没有正式 Goal Contract
→ 就不是 Agent

没有持久 Run 和状态机
→ 就不是 Agent

没有 Context Manifest
→ 就不能构建上下文

没有独立 Acceptance
→ 就不能结束任务
~~~

这些结论都不准确。它们混淆了：

~~~text
一种能力是否让系统更可靠
≠ 这种能力是否是 Agent 成立的必要条件
~~~

## 第一层：Agent 基础结构

最小 Agent 可以只有：

~~~text
用户请求或当前目标
→ 程序组织本次模型输入
→ 调用模型
→ 模型提出回答或 Tool Call
→ 程序执行允许的工具
→ 工具结果返回模型
→ 模型根据新结果继续或停止
~~~

这一层必须存在的不是某组固定对象名，而是四项职责：

1. 有一个当前要解决的问题或目标；
2. 模型能够根据当前 Context 提出下一步；
3. 外部程序能够执行工具并返回 Observation；
4. 有明确的继续或停止边界。

简单实现可以：

- 直接把用户请求当作 Goal；
- 只使用内存中的消息列表；
- 不显式创建 Run、Phase 或 Attempt；
- 每次把近期历史和工具结果重新发送给模型；
- 由模型提出完成，并由程序做一个最小检查后结束。

这类 Agent 可靠性有限，但仍然是 Agent。

## 第二层：持续 Agent 的常见工程增强

任务跨多个 Turn、时间较长或可能中断时，通常需要增加：

~~~text
持久 Thread 或任务标识
持久 Goal 与 Run State
预算、超时和取消
工作记忆或阶段摘要
Checkpoint 与恢复
有副作用工具的幂等处理
较稳定的验证步骤
~~~

这些能力解决的是“任务怎样持续、安全恢复并避免重复副作用”，而不是重新定义 Agent。

## 第三层：高可靠理想结构

当任务风险高、影响范围大或需要审计时，可以继续增加：

~~~text
正式 Goal Contract 与 revision
独立 Goal Intake
Phase、Capability 与转换守卫
带来源和版本的 Context Package
Context Manifest
Candidate 与 Artifact 身份
Verification Obligation
Evidence
独立 Acceptance
完整审计和人工批准
~~~

这是一套高可靠控制架构的能力清单。具体系统可以合并组件，也可能只选择其中一部分。

## 同一个问题在三层中的不同做法

| 问题 | 基础 Agent | 持续 Agent | 高可靠理想结构 |
|---|---|---|---|
| Goal | 当前用户请求 | 持久 Objective | Goal Contract、Scope、revision |
| 运行身份 | 当前循环 | Run ID 与状态 | Phase、Attempt、状态机 |
| Context | 近期消息和工具结果 | 状态、摘要与按需检索 | 来源、版本、Package 与 Manifest |
| 工具 | 名称和参数校验 | 权限、预算、重试 | 阶段能力、Approval、审计 |
| 恢复 | 重新开始或人工继续 | Checkpoint、幂等 | 外部现实核对与受控 Resume |
| 完成 | 模型停止加最小检查 | 指定测试或规则 | Candidate、Evidence、Acceptance |

## “理想状态”不等于所有组件都拆成服务

逻辑职责需要分清，不代表物理部署必须复杂。

例如一个本地单进程 Agent 也可以在同一程序中完成：

~~~text
循环控制
状态存储
上下文选择
工具执行
验证
~~~

高可靠的核心是权威和校验边界，而不是微服务数量。

## 每篇笔记怎样标记

后续笔记遵循：

- **基础 Agent 结构**：理解 Agent 因果链所必需；
- **持续 Agent 常见增强**：长任务、跨 Turn 或可恢复任务需要；
- **高可靠理想结构（可选）**：高风险、审计或独立控制场景需要。

如果一篇全部属于高可靠设计，就直接标为选读，不在正文中假装它是基础 Agent 的组成部分。只有确实同时包含两层内容的笔记，才在正文中分别展开。

## 判断某项能力是否过度

可以连续问：

1. 没有它，模型—工具—Observation 循环还能否成立？
2. 它解决的是 Agent 定义问题，还是可靠性、恢复、审计问题？
3. 当前学习目标是理解基础结构，还是准备生产实现？
4. 这项能力是否会改变主线因果关系？
5. 是否可以在需要时再增加，而不妨碍先理解下一组件？

如果没有它仍能形成受控行动循环，它通常不应进入基础必读定义。

## 停止点

记住：

> 基础结构解释 Agent 为什么能够行动；理想结构解释怎样让这种行动更持久、更安全、更可验证。

下一篇：[[00-Agent框架速览概览|Agent 框架速览概览]]。
