---
type: implementation-note
module: 3
status: complete
audience: non-specialist
parent: "[[00-Goal-Intake搭建概览|Goal Intake 搭建概览]]"
tags: [goal-schema, state-flow, implementation]
---

# Goal Schema 与最小状态流

> [!summary]
> Goal Schema 把任务契约变成程序可以校验、保存和版本化的数据；Schema 只保证结构合法，语义质量仍需要用户确认和后续检查。

## 一份说明性 Schema

下面不是要求照抄的最终实现，而是帮助理解最小对象：

```text
Goal
  id
  revision
  objective
  successCriteria[]
    id
    description
    required
  scope
    projectOrEnvironment
    includedAreas[]
    constraints[]
  nonGoals[]
  assumptions[]
  openQuestions[]
  sourceRequestRefs[]
  confirmationRefs[]
  status
  createdAt
  updatedAt
```

## 最小结构校验

程序可以自动检查：

- Objective 非空；
- 至少存在一条 required success criterion；
- 每条标准有稳定 ID；
- Scope 绑定明确项目或环境；
- 字段大小和格式受限；
- revision 单调增加；
- confirmation 绑定当前 Draft revision。

## 程序不能只靠 Schema 判断什么

```text
“系统正常运行”
```

是合法字符串，却可能是很差的成功条件。Schema 很难独自判断：

- 是否覆盖真实业务路径；
- 是否遗漏关键风险；
- 用户是否有权做出决定；
- 条件是否真的可以获得证据；
- 模型是否偷偷加入业务假设。

因此还需要语义检查和用户确认。

## 最小状态流

```text
DRAFT
→ NEEDS_CLARIFICATION
→ READY_FOR_CONFIRMATION
→ CONFIRMED
→ PROMOTED_TO_GOAL

任意草稿状态
→ ABANDONED
```

这是一种说明性设计。正式 Goal 可以拥有另一套生命周期，例如：

```text
ACTIVE
→ WAITING_FOR_INPUT / BLOCKED
→ CLOSED / CANCELLED
```

不要把 Draft 状态和正式 Goal 运行状态放进同一个含义模糊的 `status`。

## 创建边界

```text
Draft Assistant
→ 生成提案

Confirmation Gateway
→ 记录用户决定

Goal Manager
→ 校验并创建 Goal identity 和 revision

Store
→ 原子保存 Goal 与审计事件
```

## 第一版接口可以很简单

```text
createDraft(rawRequest, projectRef)
answerClarification(draftId, questionId, answer)
confirmDraft(draftId, draftRevision)
promoteGoal(draftId, confirmedRevision)
```

真正编码时还需要定义认证、错误、并发、幂等和存储事务。本页只确定职责和最小状态边界。
