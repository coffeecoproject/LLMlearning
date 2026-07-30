---
type: implementation-overview
module: 3
status: complete
audience: non-specialist
parent: "[[00-目标与任务契约概览|目标与任务契约概览]]"
tags: [goal-intake, architecture, implementation]
---

# Goal Intake 搭建概览

> [!summary]
> Goal Intake 是正式 Agent Run 之前的需求形成系统：它允许模型帮助起草和提问，但必须把草稿权威、用户决定和正式 Goal 创建分开。

## 什么时候需要搭建

Goal Intake 是高可靠增强层，不是每个 Agent 的前置依赖。一次性、低风险且边界清楚的任务可以直接执行；当需求模糊、执行周期长、会产生较大副作用，或者必须形成可审计验收边界时，再引入这一层。

## 推荐的逻辑组件

```text
CLI / GUI / API
→ Intake Coordinator
   ├── Raw Request Store
   ├── Goal Draft Assistant（可调用 LLM）
   ├── Ambiguity Checker
   ├── Bounded Read-only Explorer（可选）
   └── Human Confirmation Gateway
            ↓
        Goal Manager
            ↓
     Formal Goal Store
            ↓
      Agent Runtime
```

这些组件可以先写在同一个程序中。这里按职责拆分，不代表必须部署多个服务器。

## 最小版本需要什么

第一版只需要完成：

```text
1. 保存用户原话
2. 调用模型生成结构化 Goal Draft
3. 展示 Objective、Criteria、Scope、Non-goals 和 Open Questions
4. 用户编辑或确认
5. 程序做结构校验
6. Goal Manager 创建正式 Goal
```

第一版暂时不需要：

- 自动遍历整个项目；
- 完整 Fact Graph；
- 自动推断所有验收测试；
- 多 Agent 产品访谈；
- 自动批准业务取舍；
- 复杂向量数据库。

## 为什么需要独立 Intake 身份

正式 Worker 通常绑定 `GoalId` 和 `RunId`，但 Goal Intake 开始时还没有正式 Goal。

因此 Intake 需要自己的草稿身份，例如：

```text
IntakeSessionId
RawRequestId
GoalDraftId
DraftRevision
```

这些对象可以持久化，也可以在最小原型中短期保存；但不能冒充正式 Goal 权威。

## 模型的输出合同

建议要求模型输出候选结构，而不是一段无法稳定解析的长文本：

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

系统必须把候选、假设和用户已确认事实分开。

## 项目预探索边界

如果允许 Intake 读取项目，应采用：

- 只读工具；
- 明确项目路径；
- 有限文件和调用预算；
- 不创建 Candidate；
- 不修改源代码；
- 观察结果标注来源；
- 只用于提出更准确的问题。

深度业务发现仍属于正式 Goal 创建后的 Discovery。

## 搭建顺序

1. 先完成无 LLM 的手动 Goal 表单；
2. 再让 LLM 生成 Draft；
3. 加入用户确认和审计；
4. 加入有限只读探索；
5. 最后连接 revision、Fact 和 Decision 系统。

下一篇：[[01-歧义识别澄清与用户确认|歧义识别、澄清与用户确认]]。
