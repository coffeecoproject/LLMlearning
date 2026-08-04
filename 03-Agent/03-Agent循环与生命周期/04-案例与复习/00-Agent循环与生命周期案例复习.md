---
type: review-note
module: 3
learning_layer: persistent-high-reliability
status: complete
audience: non-specialist
parent: "[[00-Agent循环与生命周期概览|Agent 循环与生命周期概览]]"
tags: [agent-loop, lifecycle, case-study, review]
---

# Agent 循环与生命周期案例复习

> [!summary]
> 本案例用一次代码修复 Run 串联 Goal、Phase、Attempt、用户 Turn、Model Call、Tool Step、Observation 和验收，帮助区分产品交互、模型调用与系统状态推进。

> [!note] 整篇层级：高可靠 Run 案例（选读）
> 本案例故意使用 Phase、Attempt、持久状态和验收来展示完整控制结构；基础 Agent 只需保留模型—工具—Observation 循环。

## 起点

正式 Goal：

```text
修复重复支付回调造成的重复扣款；
重复事件只能扣款一次，首次支付和并发测试必须通过。
```

Runtime 创建：

```text
Run R1
goalRevision = 1
state = READY
turnBudget = 20
toolCallBudget = 40
```

## DISCOVERY Phase

```text
Attempt D1

Thread C1
└── 用户 Turn U1：修复重复支付回调

Model Call M1
→ 模型请求搜索支付入口

Step 1
→ Runtime 校验只读权限并执行搜索

Observation 1
→ 找到 callback.ts 和 ledger.ts

Model Call M2
→ 模型请求读取两个文件

Step 2
→ Runtime 读取文件

Observation 2
→ 发现两个路径都可能写入账本
```

模型提出根因假设后，Runtime 保存 Proposal，并根据阶段守卫进入 PLAN，而不是让模型直接写状态。

## IMPLEMENT Phase

```text
Attempt I1
→ Model Call M3 请求修改
→ Runtime 执行修改并形成 Candidate C1
→ 模型提交 Completion Request
```

Completion Request 不会关闭 Run。Runtime 进入 VERIFYING。

## 第一次验证失败

```text
并发重复回调测试失败
→ 失败结果成为 Observation 和验证证据
→ Acceptance 返回 REPAIR
→ C1 被标记为失败版本
→ 创建新的 IMPLEMENT Attempt I2 和 Candidate C2
```

Runtime 把失败证据放入新 Attempt 的上下文，但不把整个旧对话当作权威状态。

## 第二次实现和验收

```text
Attempt I2
→ 新的 Model Call 根据失败证据提出修复
→ Runtime 执行修改
→ 提交新的 Completion Request

VERIFYING
→ 重复事件测试通过
→ 首次支付测试通过
→ 并发测试通过
→ Evidence 全部绑定 C2

Acceptance = ACCEPT
→ Run R1 进入 COMPLETED
→ Codex 或客户端产生最终 Agent Message
→ 用户 Turn U1 结束
```

这里为了集中展示一个完整闭环，假设调查、实现和验证都发生在同一个用户 Turn 中。真实高可靠系统也可以让一个 Run 跨越多个用户 Turn，或者在不同 Phase 使用不同 Thread 和 Worker。

## 如果中途进程崩溃

假设 Tool Call 已经发送，但 Runtime 没收到结果：

```text
重启
→ 加载 R1 和未完成 Step
→ 检查工具调用的 commandId
→ 核对动作是否已经发生
→ 记录 Reconciliation
→ 决定接收原结果、重试或阻塞
```

它不能只根据最后一条模型消息重新执行。

## 关键对象复盘

| 对象 | 案例中的作用 |
|---|---|
| Goal | 定义重复扣款修复和成功条件 |
| Run R1 | 本次完整受控执行 |
| Phase | 限制探索、实现和验证阶段能力 |
| Attempt I1/I2 | 区分失败实现和修复实现 |
| Thread C1 | 承载持续对话和执行事件 |
| 用户 Turn U1 | 从用户提出任务到本次最终 Agent Message 的完整处理 |
| Model Call M1/M2/M3 | Agent Runtime 对模型服务的一次请求和响应 |
| Item | Turn 中的用户消息、工具执行、结果和 Agent 消息等记录 |
| Step | 搜索、读取、修改、测试等 Runtime 动作 |
| Observation | 文件结果、错误和测试结果 |
| Completion Request | 模型申请开始验收 |
| Acceptance | 根据证据决定是否完成 |

## 理解检查

1. 为什么 Model Call M2 结束后，用户 Turn 和 Run 都仍然没有完成？
2. 为什么第一次验证失败需要新的 Attempt？
3. 为什么 C1 的测试结果不能自动证明 C2？
4. 进程崩溃后为什么要先核对外部现实？
5. 哪些转换由模型提出，哪些只能由 Runtime 提交？

## 专题完成标准

- 能说明 Agent Loop 为什么位于 LLM 外部；
- 能区分 Goal、Run、Phase、Attempt、Step 与 Thread、Turn、Item、Model Call；
- 能画出继续、等待、取消、失败和完成的状态路径；
- 能解释状态守卫、预算和事件协议的作用；
- 能说明 Completion Request 为什么不等于 Acceptance。

下一专题：[[00-运行状态与持久化概览|运行状态与持久化概览]]。
