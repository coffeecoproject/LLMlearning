---
type: optional-architecture-overview
module: 3
learning_layer: persistent-high-reliability
status: complete
audience: non-specialist
parent: "[[00-目标与任务契约概览|目标与任务契约概览]]"
tags: [goal-intake, high-reliability, optional-pattern, architecture]
---

# Goal Intake 高可靠扩展概览

> [!summary]
> Goal Intake 是一种可选的高可靠需求形成模式：它在正式 Agent Run 之前帮助形成、澄清和确认 Goal Contract，但不是 Agent 成立的必要组件，也不是行业统一规定的标准名称。

## 先确定它不是什么

Goal Intake 不等于所有 Agent 都必须拥有的“第一阶段”。不同 Agent 可以采用不同深度：

```text
最小 Agent
→ 直接把用户请求作为当前 Goal

持续 Agent
→ 在同一个 Thread 中理解需求，并持久化一个 Objective

高可靠控制型 Agent
→ 可选地把需求形成、用户确认和正式执行分开
```

只有第三种情况才需要这里描述的完整 Goal Intake。

## 它一定属于 Agent Runtime 吗

不一定。Goal Intake 可以位于：

```text
用户界面或业务应用
→ 通过表单和对话帮助用户明确需求

Agent Runtime 内部
→ 作为正式 Run 之前的受控模式

独立编排层
→ 先生成并确认任务契约，再把 Goal 交给执行 Agent
```

物理上可以写在同一个程序中，逻辑上则要区分：草稿是谁提出的、业务含义由谁确认、正式 Goal 由谁创建。

## 什么时候值得增加

一次性、低风险且边界清楚的任务通常不需要独立 Intake。当出现以下情况时，它才可能值得增加：

- 用户需求明显模糊；
- 执行周期很长；
- 会产生较大或不可逆副作用；
- 多个角色需要确认业务含义；
- 必须形成可审计的 Scope 和验收边界；
- 错误理解目标的成本明显高于多一次确认的成本。

## 一种说明性的逻辑结构

下面是高可靠系统可以采用的设计示例，不是通用 Agent 标准，也不代表当前产品一定已经实现这些独立组件：

```text
CLI / GUI / API
→ Intake Coordinator
   ├── 保存 Raw Request
   ├── Goal Draft Assistant（可调用 LLM）
   ├── Ambiguity Checker
   ├── Bounded Read-only Explorer（可选）
   └── Human Confirmation Gateway
            ↓
        Goal Manager
            ↓
     Formal Goal Store
            ↓
      Agent Runtime / Worker
```

这些名称用于说明职责。一个产品完全可以把其中数项合并成一个界面流程或一个 Agent Loop。

## 如果选择实现，最小版本是什么

```text
1. 保存用户原话
2. 生成结构化 Goal Draft
3. 展示 Objective、Criteria、Scope、Non-goals 和 Open Questions
4. 让用户编辑或确认关键业务含义
5. 程序执行结构和权限校验
6. Goal Manager 创建正式 Goal
```

第一版不需要自动遍历整个项目、完整 Fact Graph、多 Agent 产品访谈、复杂向量数据库，也不应让模型自动批准业务取舍。

## 草稿为什么不能直接成为正式 Goal

如果 Intake 与正式执行分开，草稿阶段可以使用自己的身份：

```text
IntakeSessionId
RawRequestId
GoalDraftId
DraftRevision
```

这些对象只是候选状态。只有经过规定的确认和校验后，Goal Manager 才能创建正式 `GoalId` 和 `GoalRevision`。

## 模型在这里负责什么

模型适合输出候选结构：

```text
GoalDraftProposal
  objectiveCandidate
  successCriteriaCandidates[]
  scopeCandidates[]
  nonGoalCandidates[]
  ambiguities[]
  clarificationQuestions[]
  assumptions[]
```

模型不能凭自己的文字确认用户真实意图。系统必须把候选、假设、项目观察、用户决定和正式字段分开。

## 项目预探索的边界

如果允许 Intake 读取项目，应优先采用：

- 只读工具；
- 明确项目路径；
- 有限文件和调用预算；
- 不创建或修改 Candidate；
- 观察结果标注来源；
- 只用于提出更准确的问题。

深入理解项目和选择实现方式通常属于正式 Goal 创建后的 Discovery 与 Planning。

## 与当前产品实现的边界

```text
Codex CLI
→ 当前以持久 Thread Goal 和同一 Agent Loop 为主
→ 没有强制独立 Goal Intake 阶段

本页的完整 Goal Intake
→ 可放在高可靠控制 Runtime 之前的未来产品入口
→ 不是对任何当前产品内部结构的默认描述
```

当前 Codex 实现的对照见 [[01-Codex-Thread-Goal与高可靠Goal契约|Codex Thread Goal 与高可靠 Goal 契约]]。

## 如果以后搭建，推荐顺序

1. 先完成无 LLM 的手动 Goal 表单；
2. 再让 LLM 生成 Draft；
3. 加入用户确认和审计；
4. 按需要加入有限只读探索；
5. 最后连接 Goal revision、Fact 和 Decision 系统。

## 阅读路线

- **只看通用 Agent 框架**：可以跳过本文件夹，直接进入 Agent Loop；
- **理解高可靠机制**：继续阅读歧义确认、Revision 和 Schema；
- **准备构建控制型 Agent**：把这些内容视为可选产品设计，而不是 Agent 固定模板。

下一篇：[[01-歧义识别澄清与用户确认|歧义识别、澄清与用户确认]]。
