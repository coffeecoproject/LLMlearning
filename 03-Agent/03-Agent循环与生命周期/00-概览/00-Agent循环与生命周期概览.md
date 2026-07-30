---
type: topic-overview
module: 3
status: complete
audience: non-specialist
parent: "[[00-Agent模块大纲|Agent 模块大纲]]"
tags: [agent, loop, lifecycle, state-machine]
---

# Agent 循环与生命周期概览

> [!summary]
> Agent Loop 是外部程序反复进行“读取状态、构建上下文、调用模型、处理动作、接收反馈、更新状态和判断停止”的控制循环；LLM 只参与其中的决策生成环节。

## 在 Agent 主线中的位置

```text
任务目标（可以是用户请求、Thread Goal 或正式 Goal Contract）
→ 创建 Run
→ Agent Runtime 启动循环
→ 模型提出回答或动作
→ Runtime 执行并记录 Observation
→ 再次循环
→ 等待、阻塞、失败、取消或进入验收
```

模型一次生成结束，只表示一次模型交互结束。整个 Run 是否继续、重试或完成，由 Agent Runtime 根据状态和策略判断。

## 最小控制循环

```text
读取当前权威状态
→ 判断是否允许继续
→ 为本轮构建 Context Package
→ 调用 LLM
→ 解析并校验模型结果
→ 执行允许的动作
→ 记录 Observation 和状态变化
→ 判断下一状态
→ 继续下一轮或停止
```

## 本专题的核心对象

| 对象 | 最小理解 |
|---|---|
| Goal | 整个运行要满足的目标与完成边界 |
| Run | 围绕一个 Goal 启动的一次受控运行 |
| Phase | Run 中具有特定目的、权限和退出条件的阶段 |
| Turn | Agent 与模型的一轮请求和响应 |
| Step | Runtime 记录的一个处理步骤或动作 |
| Attempt | 对某个阶段或任务的一次有界执行尝试 |
| Observation | 工具、环境或用户返回的真实结果 |

这些名称不是所有框架统一采用的标准。简单 Agent 可能只显式记录 Thread 和 Turn；可靠长任务通常需要逐步建立等价的身份和边界，避免只靠聊天顺序推测状态。

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

1. [[00-Agent-Loop到底是什么|Agent Loop 到底是什么]]；
2. [[01-一次Run怎样从开始走到结束|一次 Run 怎样从开始走到结束]]。

### 理解机制

1. [[00-运行对象与状态机概览|运行对象与状态机概览]]；
2. [[01-Goal-Run-Phase-Turn-Step与Attempt|Goal、Run、Phase、Turn、Step 与 Attempt]]；
3. [[02-Observe-Decide-Act循环|Observe–Decide–Act 循环]]；
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

## 与相邻专题的边界

- 本专题负责循环身份、状态和合法转换；
- [[00-状态与持久化概览|状态与持久化]]详细解释这些状态怎样可靠保存；
- [[00-工具与环境交互概览|工具与环境交互]]详细解释动作怎样真实执行；
- [[00-反馈重试与恢复概览|反馈、重试与恢复]]详细解释失败后的重试和外部现实核对；
- [[00-验证证据与验收概览|验证、证据与验收]]决定任务是否真的满足 Goal。

## 框架停止点

如果你能说明“模型不会自己持续运行；Agent Runtime 用 Run 状态反复触发模型和工具，并在明确条件下继续、等待或停止”，本专题在框架层已经完成。
