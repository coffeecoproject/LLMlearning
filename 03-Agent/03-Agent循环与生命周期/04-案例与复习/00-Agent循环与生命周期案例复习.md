---
type: review-note
module: 3
status: complete
audience: non-specialist
parent: "[[00-Agent循环与生命周期概览|Agent 循环与生命周期概览]]"
tags: [agent-loop, lifecycle, case-study, review]
---

# Agent 循环与生命周期案例复习

> [!summary]
> 本案例用一次代码修复 Run 串联 Goal、Phase、Attempt、Turn、Tool Step、等待、修复和验收，帮助区分模型生成与系统状态推进。

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

Turn 1
→ 模型请求搜索支付入口

Step 1
→ Runtime 校验只读权限并执行搜索

Observation 1
→ 找到 callback.ts 和 ledger.ts

Turn 2
→ 模型请求读取两个文件

Observation 2
→ 发现两个路径都可能写入账本
```

模型提出根因假设后，Runtime 保存 Proposal，并根据阶段守卫进入 PLAN，而不是让模型直接写状态。

## IMPLEMENT Phase

```text
Attempt I1
→ 模型修改 Candidate C1
→ 模型提交 Completion Request
```

Completion Request 不会关闭 Run。Runtime 进入 VERIFYING。

## 第一次验证失败

```text
并发重复回调测试失败
→ Acceptance 返回 REPAIR
→ C1 被标记为失败版本
→ 创建新的 IMPLEMENT Attempt I2 和 Candidate C2
```

Runtime 把失败证据放入新 Attempt 的上下文，但不把整个旧对话当作权威状态。

## 第二次实现和验收

```text
Attempt I2
→ 模型根据失败证据修改
→ 提交新的 Completion Request

VERIFYING
→ 重复事件测试通过
→ 首次支付测试通过
→ 并发测试通过
→ Evidence 全部绑定 C2

Acceptance = ACCEPT
→ Run R1 进入 COMPLETED
```

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
| Turn | 每一次模型请求和响应 |
| Step | 搜索、读取、修改、测试等 Runtime 动作 |
| Observation | 文件结果、错误和测试结果 |
| Completion Request | 模型申请开始验收 |
| Acceptance | 根据证据决定是否完成 |

## 理解检查

1. 为什么 Turn 2 结束后 Run 仍然没有完成？
2. 为什么第一次验证失败需要新的 Attempt？
3. 为什么 C1 的测试结果不能自动证明 C2？
4. 进程崩溃后为什么要先核对外部现实？
5. 哪些转换由模型提出，哪些只能由 Runtime 提交？

## 专题完成标准

- 能说明 Agent Loop 为什么位于 LLM 外部；
- 能区分 Goal、Run、Phase、Turn、Step 和 Attempt；
- 能画出继续、等待、取消、失败和完成的状态路径；
- 能解释状态守卫、预算和事件协议的作用；
- 能说明 Completion Request 为什么不等于 Acceptance。

下一专题：[[00-状态与持久化概览|状态与持久化概览]]。
