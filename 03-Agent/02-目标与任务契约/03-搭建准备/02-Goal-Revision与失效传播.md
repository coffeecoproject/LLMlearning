---
type: implementation-note
module: 3
status: complete
audience: non-specialist
parent: "[[00-Goal-Intake搭建概览|Goal Intake 搭建概览]]"
tags: [goal-revision, invalidation, versioning]
---

# Goal Revision 与失效传播

> [!summary]
> 当 Objective、成功条件或 Scope 发生实质变化时，系统需要创建新的 Goal revision，并重新判断旧计划、上下文、产物和证据还能否使用。

## 为什么不能直接覆盖

假设 Goal v1 是：

```text
阻止未来的重复扣款，不处理历史订单。
```

用户后来要求：

```text
同时识别历史重复扣款并生成退款任务。
```

如果直接覆盖原字段，系统会失去：

- 原来的执行依据；
- 哪些工作发生在变更之前；
- 旧证据证明的是哪个目标；
- 为什么计划和权限发生变化。

## Revision 主线

```text
Goal v1
→ 收到新的用户决定或关键事实
→ 形成 Goal v2 Draft
→ 用户确认变化
→ Goal Manager 创建 Goal v2
→ 触发依赖项重新评估
```

## 哪些变化通常需要新 revision

- Objective 改变；
- Required Success Criterion 增删或语义改变；
- Scope 扩大或缩小；
- Non-goal 被移除或新增；
- 会改变执行和验收的业务决定发生变化。

仅仅更新运行状态、记录新 Observation 或修正文字错别字，不一定需要 Goal revision。

## 失效不等于全部删除

Goal 改变后，系统应逐类判断：

| 依赖对象 | 可能处理 |
|---|---|
| Plan | 重新生成或保留未受影响部分 |
| Context Manifest | 标记过期并重新编译 |
| Task | 取消、修订或新增 |
| Candidate | 判断是否覆盖新范围 |
| Evidence | 仅保留仍绑定有效标准的证据 |
| Acceptance | 旧决定不能自动批准新 Goal |

历史记录仍应保留用于审计，只是不能继续作为当前权威。

## 防止旧结果误推进

每个重要请求和结果都应携带它所依据的：

```text
goalId
goalRevision
runId
必要时还有 planRevision / candidateId / contextManifestId
```

如果 Worker 针对 v1 完成时，当前 Goal 已经是 v2，系统可以保存它的结果，但不能让这个旧结果直接推进 v2。

## 搭建原则

```text
变化先形成新权威版本
→ 再显式传播失效
→ 最后允许新执行继续
```

不能依赖模型在长对话中“记住目标后来改过”。
