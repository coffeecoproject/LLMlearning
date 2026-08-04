---
type: review-note
module: 3
learning_layer: cross-layer
status: planned
audience: non-specialist
parent: "[[00-Agent模块大纲|Agent 模块大纲]]"
tags: [agent, boundary, review, architecture]
---

# Agent 边界与复习概览

> [!summary]
> Agent 的能力来自模型与外部控制系统的组合：模型负责生成候选决策，控制系统负责目标、状态、工具、权限、反馈、恢复和验收；系统复杂度增加并不会自动消除模型的不确定性。

> [!info] 层级定位
> 复习时先检查基础行动循环是否成立，再单独评价持久化、安全、恢复和验收等级；不能用一套高可靠清单反向否定所有简单 Agent。

## 最终复习主线

```text
Raw Request
→ Goal Contract
→ Agent Run
→ Context Package
→ Model Decision
→ Authorized Tool Action
→ Observation
→ State Update
→ Retry / Replan / Continue
→ Evidence
→ Acceptance
→ Closeout
```

## 本专题最终要回答

- Agent 与普通模型调用、Workflow 有什么区别；
- Goal、Plan、State、Context 和 Memory 怎样区分；
- Tool Call 为什么只是动作提案；
- Agent Loop 为什么由 Runtime 而不是 LLM 自己维持；
- 失败后怎样防止重复副作用并恢复；
- Verification、Evidence 与 Acceptance 怎样形成完成边界；
- Agent Runtime 与模型 Runtime 为什么不是同一层；
- 什么时候不需要 Agent，什么时候不应增加多 Agent；
- Agent 运行记忆与持续学习为什么仍要分开。

## 安全停止点

只看框架时，能够独立画出上面的主线，并对每一步说明“谁负责、保存什么、怎样失败、由谁确认”，就可以结束 Agent 框架学习。

## 进入后续模块

```text
Agent
→ 已解决一次或一组任务怎样受控执行

持续学习
→ 继续研究跨任务经验怎样积累、选择、更新和遗忘
```

本页将在前面专题完成后补充完整案例、常见误解和综合理解检查。
