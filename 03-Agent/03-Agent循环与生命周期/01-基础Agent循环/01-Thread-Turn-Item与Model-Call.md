---
type: mechanism-note
module: 3
learning_layer: basic
status: complete
audience: non-specialist
parent: "[[00-运行对象与状态机概览|运行对象与状态机概览]]"
tags: [thread, turn, item, model-call, codex]
---

# Thread、Turn、Item 与 Model Call

> [!summary]
> 在 Codex 中，Thread 是持续对话，Turn 是从一次用户输入到最终 Agent 消息的完整处理过程，Item 是 Turn 中可记录和展示的事件；Model Call 则是 Agent Runtime 在这个过程中对模型发起的一次实际调用。

## 为什么需要单独区分

在普通聊天中，“一轮”很容易被理解成一次问答；在 Agent 系统中，一次用户请求可能触发多次模型调用和多次工具执行。如果都叫 Turn，就无法判断究竟是：

- 用户发起的一次完整处理；
- 对模型的一次请求和响应；
- 还是过程中发生的一条事件。

因此，本模块统一使用下面的说法：

| 名称 | 本模块中的含义 |
|---|---|
| Thread | 一整个持续对话或任务交互容器 |
| Turn | 从一次用户输入开始，到本次最终 Agent Message 结束 |
| Item | Turn 中一条可单独记录、展示和追踪的事件 |
| Model Call | Agent Runtime 对模型服务的一次请求和响应 |
| Tool Call | 模型提出的一次工具请求 |
| Tool Result | 工具实际执行后返回的结果 |

## Codex 中的结构

```text
Thread
├── Turn 1
│   ├── Item：用户消息
│   ├── Model Call 1
│   │   └── 产生 Item：工具调用
│   ├── Item：工具执行及结果
│   ├── Model Call 2
│   │   └── 产生 Item：继续读取文件
│   ├── Item：新的工具执行及结果
│   └── Model Call 3
│       └── 产生 Item：最终 Agent Message
└── Turn 2
    └── 用户下一次输入引发的完整处理
```

`Model Call` 不是 Codex App Server 的顶层对话对象，也不一定在界面中显示为一个独立 Item。它是 Agent Loop 内部真实发生的一次模型交互；模型交互产生的消息、推理、工具调用等内容，才会映射成相应 Item。

## 一个 Turn 为什么会调用模型多次

用户输入：

```text
查看退款功能是怎样实现的，不要修改代码。
```

同一个 Turn 可能发生：

```text
Item：用户消息
→ Model Call 1：模型请求搜索 refund
→ Item：搜索工具执行与结果
→ Model Call 2：模型请求读取两个文件
→ Item：文件读取结果
→ Model Call 3：模型根据代码继续检查测试
→ Item：测试文件读取结果
→ Model Call 4：模型输出最终说明
→ Item：最终 Agent Message
→ Turn Completed
```

所以：

```text
一次 Model Response 完成
≠ 整个 Turn 已经完成

模型返回 Tool Call
→ 当前 Model Call 已结束
→ Agent Runtime 执行工具
→ 再发起新的 Model Call
→ 仍然属于同一个 Turn
```

只有当 Agent Loop 不再需要工具或其他继续动作，并产生本次最终 Agent Message 时，当前 Turn 才结束。

## Item 到底记录什么

常见 Item 包括：

- 用户消息；
- Agent 消息；
- 推理摘要；
- Shell 命令；
- 文件修改；
- 工具调用和执行结果；
- 审批请求；
- 计划、Diff 或其他结构化产物。

一个流式 Item 还可能经历：

```text
started
→ 若干 delta
→ completed
```

例如命令执行开始时先产生 `started`，执行日志通过多个 `delta` 到达，最后由 `completed` 给出退出状态和完整结果。流式片段适合界面展示，但不能把每个 Token 或 delta 都当成一次权威状态转换。

## Item 与 Observation 的关系

两者回答不同问题：

```text
Item
= 系统怎样记录和展示一件事

Observation
= 这条结果怎样成为 Agent 下一步决策的环境反馈
```

例如测试结果：

```text
从协议结构看
→ 它是一个工具执行结果 Item

从 Agent Loop 看
→ 它是一次 Observation

从验收系统看
→ 绑定来源和产物版本后，还可能成为 Evidence
```

因此，一个工具结果可以同时是 Item 和 Observation，但用户消息、工具请求、最终回答等 Item 通常不都称为 Observation。

## 与 Run 的关系

`Thread / Turn / Item` 更接近产品交互和协议对象；`Goal / Run / Phase / Attempt` 更接近任务执行与控制对象。它们不是一条所有系统都必须采用的固定父子树。

```text
简单 Codex 请求
→ 一个 Turn 可以近似看作本次请求的一次运行

持续 Goal
→ 一个 Run 可能跨越多个 Turn

高可靠外部控制系统
→ 一个 Run 还可能在不同 Phase 使用不同 Thread 或 Worker
```

下一篇：[[02-Observe-Decide-Act循环|Observe–Decide–Act 循环]]。

## 开放实现观察

以上 Codex 对象关系核对自 OpenAI Codex 开源仓库，版本为提交 `6219b7c40fc9c702c0aef9964e72b492558f60e4`，核对日期为 2026-07-30：

- [App Server 的 Thread、Turn、Item 定义](https://github.com/openai/codex/blob/6219b7c40fc9c702c0aef9964e72b492558f60e4/codex-rs/app-server/README.md#L66-L83)；
- [同一 Turn 可以发送一次或多次 Responses API 请求](https://github.com/openai/codex/blob/6219b7c40fc9c702c0aef9964e72b492558f60e4/codex-rs/core/src/client.rs#L1-L19)；
- [工具结果写回历史并触发后续模型调用](https://github.com/openai/codex/blob/6219b7c40fc9c702c0aef9964e72b492558f60e4/codex-rs/core/src/session/turn.rs#L2118-L2142)。
