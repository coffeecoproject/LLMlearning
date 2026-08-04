---
type: topic-overview
module: 3
learning_layer: cross-layer
status: complete
audience: non-specialist
parent: "[[00-Agent模块大纲|Agent 模块大纲]]"
tags: [agent, loop, lifecycle, state-machine]
---

# Agent 循环与生命周期概览

> [!summary]
> Agent Loop 是外部程序反复进行“读取状态、构建上下文、调用模型、处理动作、接收反馈、更新状态和判断停止”的控制循环；LLM 只参与其中的决策生成环节。

## 本专题的两层边界

### Agent 基础结构

只需理解外部程序反复执行“模型决策 → 工具行动 → 环境反馈”，并有继续或停止条件。简单 Agent 可以没有显式 Run、Phase、Attempt 或状态机。

### 持续与高可靠理想结构（选读）

Run 身份、Phase、Attempt、状态转换守卫、预算、中断恢复和独立 Acceptance 用于长任务、恢复、权限分层与审计。

## 基础 Agent 主线中的位置

```text
用户请求
→ 程序组装本次模型输入
→ 模型返回最终回答或 Tool Call
→ 若是 Tool Call，程序执行工具并取得结果
→ 把结果加入下一次模型输入
→ 模型继续决策，直到给出最终回答或触发停止条件
```

模型一次生成结束，只表示一次 Model Call 结束。外部程序根据结果决定执行工具、再次调用模型，还是结束当前用户 Turn。

## 持续与高可靠主线（选读）

```text
任务目标（Thread Goal 或 Goal Contract）
→ 创建或恢复 Run
→ 读取权威状态并构建 Context Package
→ 调用模型并校验决策
→ 执行获准动作、记录 Observation 与状态变化
→ 按预算、状态机和验收规则继续、等待、恢复或停止
```

第二条链路是在基础循环之外增加持久化、授权、恢复与验收控制，不是 Agent Loop 的定义本身。

## 本专题的三组对象

```text
任务执行对象
→ Goal、Run、Phase、Attempt、Step

产品交互与协议对象
→ Thread、Turn、Item、Model Call

循环反馈对象
→ Action、Tool Result、Observation
```

它们会互相引用，但不构成所有 Agent 都必须照搬的一棵父子树。尤其在 Codex 中：

```text
Turn
= 从一次用户输入到最终 Agent Message

一个 Turn 内
= 可以发生多次 Model Call、工具执行和 Observation
```

简单 Agent 可能只显式记录 Thread、Turn 和 Item；高可靠长任务则会在产品协议之外增加 Goal、Run、Phase、Attempt、Evidence 等控制对象。

## 实现可以合并，也可以拆分

```text
Codex 式持续 Agent
→ Thread Goal、计划、工具循环和完成审计主要在同一线程运行

高可靠控制型 Agent
→ Goal Contract、Run、Phase、Worker、Evidence 和 Acceptance 分别保存和控制
```

二者都包含 Agent Loop。区别不是“有没有 Agent”，而是职责是否被外部化、持久化和独立授权。

## 四层阅读结构

### 只看框架

只读 [[00-Agent-Loop到底是什么|Agent Loop 到底是什么]]，先掌握程序如何把多次 Model Call 与工具执行连成循环。

### 理解机制

基础 Agent 先读：

1. [[01-Thread-Turn-Item与Model-Call|Thread、Turn、Item 与 Model Call]]；
2. [[02-Observe-Decide-Act循环|Observe–Decide–Act 循环]]。

持续与高可靠 Agent 再选读：

1. [[00-运行对象与状态机概览|运行对象与状态机概览]]；
2. [[01-一次Run怎样从开始走到结束|一次 Run 怎样从开始走到结束]]；
3. [[02-Goal-Run-Phase-Step与Attempt|Goal、Run、Phase、Step 与 Attempt]]；
4. [[03-状态转换与守卫|状态转换与守卫]]；
5. [[04-等待取消失败与完成|等待、取消、失败与完成]]。

### 为搭建做准备

1. [[00-Agent-Loop搭建概览|Agent Loop 搭建概览]]；
2. [[01-控制循环与事件协议|控制循环与事件协议]]；
3. [[02-预算超时与无限循环防护|预算、超时与无限循环防护]]；
4. [[03-中断恢复与幂等入口|中断、恢复与幂等入口]]；
5. [[04-Completion-Request与Acceptance边界|Completion Request 与 Acceptance 边界]]。

### 案例与复习

- [[00-Agent循环与生命周期案例复习|Agent 循环与生命周期案例复习]]。
- [[01-Codex-CLI查看功能的完整运行链路|Codex CLI 查看功能的完整运行链路]]。

## 与相邻专题的边界

- 本专题负责循环身份、状态和合法转换；
- [[00-运行状态与持久化概览|运行状态与持久化]]详细解释这些状态怎样可靠保存；
- [[00-工具与环境交互概览|工具与环境交互]]详细解释动作怎样真实执行；
- [[00-Observation反馈重试与恢复概览|Observation、反馈、重试与恢复]]详细解释工具结果怎样推动下一轮，以及失败后的重试和外部现实核对；
- [[00-验证证据与验收概览|验证、证据与验收]]决定任务是否真的满足 Goal。

## 框架停止点

如果你能说明“模型不会自己持续运行；外部程序根据模型结果执行工具、返回 Observation，并决定再次调用还是结束”，本专题在框架层已经完成。Run、Phase、状态机、恢复和独立 Acceptance 可以按任务风险继续选读。
