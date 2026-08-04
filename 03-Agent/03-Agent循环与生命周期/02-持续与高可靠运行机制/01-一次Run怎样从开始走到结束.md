---
type: framework-note
module: 3
learning_layer: persistent-high-reliability
status: complete
audience: non-specialist
parent: "[[00-Agent循环与生命周期概览|Agent 循环与生命周期概览]]"
tags: [agent, run, lifecycle, framework]
---

# 一次 Run 怎样从开始走到结束

> [!summary]
> Run 是 Agent 围绕某个 Goal 进行的一次受控执行；它从绑定目标和策略开始，经历若干 Model Call、动作与 Observation，也可能跨越多个用户 Turn，最终进入完成、阻塞、失败或取消状态。

> [!info] 层级定位
> 显式 Run 是持续 Agent 常见的控制对象，不是最小 Agent 的必需名称。简单 Agent 可以让一次调用循环直接随当前请求开始和结束；跨 Turn、可恢复或高风险任务才更需要独立 Run 身份与状态。

## 一个高可靠受控 Run

```text
任务契约已确认
→ 创建 Run
→ 状态 READY
→ Runtime 检查前置条件
→ 状态 RUNNING
→ 执行多个 Model Call 和 Tool Step
→ 模型提交 Completion Request
→ 状态 VERIFYING
→ Acceptance 通过
→ 状态 COMPLETED
```

这是一条高可靠说明性状态链，具体系统可以合并阶段或使用不同名称。Codex Thread Goal 当前可以持续执行并在完成审计后更新状态，但不把这里的 `VERIFYING` 和独立 `Acceptance` 暴露为同样的领域对象。

## Run 开始时需要绑定什么

至少需要明确：

- Goal identity 和 revision；
- 当前项目或环境；
- 使用的 Workflow 或状态机版本；
- 权限与策略；
- 模型和工具配置；
- 时间、步骤和费用预算。

如果这些内容在运行中悄悄变化，后续结果很难判断依据是什么。

## Run 中会发生什么

```text
用户 Turn 1：用户要求查看项目功能

Model Call 1：模型要求读取项目结构
Step 1：Runtime 执行只读工具
Observation 1：返回目录结果

Model Call 2：模型要求读取两个文件
Step 2：Runtime 执行读取
Observation 2：返回代码

Model Call 3：模型形成结论并给出最终 Agent Message
用户 Turn 1 结束
```

如果用户随后要求继续修改，可以产生 Turn 2，但仍可能属于同一个持续 Goal 或外部 Run。`Turn`、`Model Call` 和 `Step` 的具体关系见 [[01-Thread-Turn-Item与Model-Call|Thread、Turn、Item 与 Model Call]]。

## Run 不只有成功和失败

```text
WAITING_FOR_USER
→ 缺少必须由用户决定的业务信息

BLOCKED
→ 当前条件下无法安全继续

FAILED
→ 本次运行发生不可恢复错误或耗尽预算

CANCELLED
→ 用户或系统主动终止

COMPLETED
→ 验收确认 Goal 已满足
```

`CANCELLED`、`BLOCKED` 和 `FAILED` 都不能包装成“已完成”。

## 模型响应结束不是 Turn 或 Run 结束

一次模型服务响应完成，只说明当前 Model Call 结束。它可能返回：

- 一个工具请求；
- 一个澄清问题；
- 一段分析；
- 一个完成申请；
- 一个无法继续的说明。

Agent Runtime 还要解析结果、执行动作、更新状态并判断下一步。只有整个用户请求的循环结束，Codex 才会报告 `Turn Completed`；即便如此，外部高可靠 Run 也可能还在等待独立 Verification 或 Acceptance。

## 框架停止点

如果你能区分“Model Call 结束”“Codex Turn 结束”和“Run 被 Acceptance 判定完成”，就可以进入下一专题。想继续理解内部对象，阅读 [[00-运行对象与状态机概览|运行对象与状态机概览]]。
