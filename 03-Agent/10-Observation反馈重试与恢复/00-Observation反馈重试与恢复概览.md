---
type: topic-overview
module: 3
learning_layer: cross-layer
status: planned
audience: non-specialist
parent: "[[00-Agent模块大纲|Agent 模块大纲]]"
tags: [agent, feedback, retry, recovery, idempotency]
---

# Observation、反馈、重试与恢复概览

> [!summary]
> Observation（观察结果）是工具和环境返回给 Agent 的真实结果；反馈让 Agent 根据它调整下一步，可靠恢复则要求系统知道哪一步已经发生、是否可以安全重做，以及从哪个权威状态继续。

> [!info] 层级定位
> 基础 Agent 只需把 Tool Result 送回下一次模型调用；错误分类、幂等、Checkpoint、Resume、熔断和外部现实核对属于持续或高可靠恢复机制。

## 它位于执行流程的哪里

```text
模型提出 Tool Call
→ Runtime 校验并执行工具
→ 工具返回 Tool Result
→ Runtime 整理成 Observation
→ 更新运行状态与上下文
→ 继续、重试、重新规划、询问用户或停止
```

`Tool Result` 更接近工具返回的原始结果；`Observation` 是 Agent 可以记录并用于下一轮决策的观察。具体系统可以把两者合并，但概念上要知道它们承担的角色。

## Item、Tool Result、Observation 与 Evidence

这四个词属于不同分类角度：

| 名称 | 回答的问题 |
|---|---|
| Item | 系统怎样记录、展示和追踪这件事 |
| Tool Result | 工具原始返回了什么 |
| Observation | 哪些环境反馈会影响 Agent 下一步决策 |
| Evidence | 哪些有来源、版本和验收条件绑定的结果可以证明某项要求 |

例如：

```text
运行并发退款测试
→ Tool Result：退出码 1 和失败日志
→ Codex Item：一次已完成的命令执行记录
→ Observation：并发情况下仍发生两次退款
→ Evidence：绑定到当前 Candidate 后，证明并发验收条件未满足
```

因此，一个工具结果可以同时是 Item、Observation，并在满足证据要求后成为 Evidence。但：

```text
Item存在
≠ 它一定是环境观察

Observation被模型看到
≠ 它已经足以完成验收
```

Codex 的 `Item` 是产品和协议对象；Observation 是通用 Agent Loop 中的语义角色。详细关系见 [[01-Thread-Turn-Item与Model-Call|Thread、Turn、Item 与 Model Call]]。

## 反馈循环

```text
Action
→ Observation
→ 分类结果
   ├── 成功，继续
   ├── 可重试错误
   ├── 需要重新规划
   ├── 需要用户决定
   └── 不可恢复失败
→ 更新状态
```

## 本专题要解决的问题

- Feedback、Observation、Error 和 Evidence 有什么区别；
- 哪些失败适合重试，哪些重试只会重复副作用；
- Retry、Repair、Replan 和 Resume 分别是什么；
- 幂等（重复请求不会重复产生业务效果）、去重和事务怎样防止重复扣款一类问题；
- Checkpoint 和审计事件怎样支持进程重启；
- 超时、取消、进程崩溃和网络断开怎样协调；
- 重试次数、时间和费用预算怎样设置边界。

## 关键边界

“再问一次模型”不是完整恢复机制。如果系统不知道工具是否已经真实执行，就必须先核对外部现实，而不是直接重复动作。

## 后续展开

1. Observation 与错误分类；
2. Retry、Repair 与 Replan；
3. 幂等键、去重和事务；
4. Checkpoint 与 Resume；
5. 外部现实核对；
6. 重试预算、熔断和人工接管。

## 阅读路线

- 只看框架：理解失败不会自动回滚；
- 理解机制：能判断一次失败应该重试、修复、重新规划还是阻塞；
- 为搭建做准备：每个有副作用的工具都必须先回答“重复执行会怎样”。

当 Agent 认为目标已经满足时，下一专题 [[00-验证证据与验收概览|验证、证据与验收]] 会解释为什么“模型声称完成”还不能直接成为最终结论。
