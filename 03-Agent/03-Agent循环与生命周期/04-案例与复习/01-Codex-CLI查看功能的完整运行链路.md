---
type: implementation-observation
module: 3
learning_layer: basic
status: complete
audience: non-specialist
parent: "[[00-Agent循环与生命周期概览|Agent 循环与生命周期概览]]"
tags: [codex, thread, turn, item, agent-loop, case-study]
---

# Codex CLI 查看功能的完整运行链路

> [!summary]
> Codex 不会先把整个工程一次性交给模型，而是在同一个用户 Turn 中多次调用模型；模型按需请求搜索和读取，本地 Codex 执行工具并把结果作为 Observation 写回历史，直到模型给出最终 Agent Message。

## 用户提出需求

用户在项目目录中输入：

```text
查看订单退款功能现在是怎么实现的，不要修改代码。
```

这会在当前 Thread 中启动一个新的 Turn。

## 物理链路

```text
用户
→ Codex CLI
→ 本地 Codex Core / Agent Runtime
→ OpenAI 模型服务
→ 模型推理 Runtime
→ LLM

LLM 请求工具
→ 模型服务返回 Tool Call
→ 本地 Codex Core 检查权限并执行工具
→ 项目文件或命令返回结果
→ Codex 把结果写回当前历史
→ 再次调用模型
```

本地 Codex Core 负责 Agent Loop；模型服务器负责 Tokenizer、模型推理、Hidden State、QKV 和 KV Cache 等模型计算。两种 Runtime 不应混为一谈。

## 当前 Turn 的展开过程

```text
Thread C1
└── Turn U1：查看退款功能
    ├── Item：User Message
    │
    ├── Model Call M1
    │   └── 模型决定搜索 refund、退款、RefundService
    ├── Item：Tool Call / Command Execution
    ├── Item：Tool Result
    │   └── Observation O1：找到 Controller、Service 和测试文件
    │
    ├── Model Call M2
    │   └── 模型决定读取 Controller 和 Service
    ├── Item：文件读取执行及结果
    │   └── Observation O2：发现调用 PaymentGateway
    │
    ├── Model Call M3
    │   └── 模型决定检查重复退款保护和测试
    ├── Item：搜索与文件读取结果
    │   └── Observation O3：存在唯一键，但缺少并发测试
    │
    └── Model Call M4
        └── Item：最终 Agent Message
            “当前退款调用链是……主要风险是……”
```

M1 到 M4 都处于同一个用户 Turn 中。模型每次只根据当前已有上下文生成；只有工具真实执行并返回结果后，下一次 Model Call 才能看到新的项目证据。

## Codex Core 在每次 Model Call 前做什么

```text
读取当前 Conversation History
→ 加入基础指令、AGENTS.md、环境和权限信息
→ 加入用户消息、以前的工具调用和结果
→ 加入本次可见工具定义
→ 构成 Prompt
→ 发送给模型服务
```

项目全部文件不会自动成为 Prompt。模型通常先用文件列表、关键词搜索、符号搜索和定向读取逐步取得相关代码。

## 模型返回 Tool Call 后发生什么

```text
模型输出 Tool Call
→ Codex Tool Router 识别工具和参数
→ 检查工具是否存在
→ 检查 Sandbox、Approval 和 Hook
→ 在本地执行搜索、读取或命令
→ 收集标准输出（stdout）、错误输出（stderr）、退出码或文件内容
→ 记录为工具结果 Item
→ 作为 Observation 进入下一次模型上下文
```

如果工具被拒绝或失败，拒绝原因和错误同样会成为下一步可以观察的结果；模型可以改用其他安全路径、询问用户或停止。

这里的 `Tool Router` 是把模型请求分发给具体工具的程序；`Sandbox` 是限制命令权限和可访问范围的隔离环境；`Approval` 是需要用户授权的检查；`Hook` 是在特定执行节点自动运行的附加检查或处理逻辑。

## Turn 什么时候结束

```text
模型仍然请求工具
→ needs_follow_up
→ Agent Loop 继续

模型不再请求工具并产生最终 Agent Message
→ 当前 Turn 完成
```

`Response Completed` 只表示一次 Model Call 的模型响应完成；`Turn Completed` 才表示本次用户请求对应的 Agent Loop 已经结束。

## 哪些内容持久，哪些内容临时

| 内容 | 生命周期 |
|---|---|
| 用户消息、工具执行 Item、最终 Agent Message | 可持久化到 Thread 历史 |
| 项目文件和 Git 状态 | 持久存在于工作区 |
| 正在等待完成的工具任务（Future）、取消信号、流式连接 | 当前进程或 Turn 的临时状态 |
| 当前 Prompt | 为某次 Model Call 临时构建 |
| Hidden State、QKV、KV Cache | 模型推理 Runtime 的临时状态 |

## 能力边界

Codex 是按需调查，不是先完整理解整个仓库。因此结果完整度仍取决于：

- 模型选择的搜索关键词是否覆盖真实入口；
- 是否继续追踪调用方、下游依赖、配置和测试；
- 工具结果是否被截断；
- 上下文是否过长；
- 模型是否过早认为证据已经足够。

高可靠外层系统可以要求固定的覆盖清单、独立验证和验收证据，但不应假装一次 Agent Loop 天然拥有完整仓库认知。

## 源码依据

以下观察基于 OpenAI Codex 开源仓库提交 `6219b7c40fc9c702c0aef9964e72b492558f60e4`，核对日期为 2026-07-30：

- [Thread、Turn 与 Item 的定义](https://github.com/openai/codex/blob/6219b7c40fc9c702c0aef9964e72b492558f60e4/codex-rs/app-server/README.md#L66-L83)；
- [Prompt 包含历史输入、工具与基础指令](https://github.com/openai/codex/blob/6219b7c40fc9c702c0aef9964e72b492558f60e4/codex-rs/core/src/client_common.rs#L16-L35)；
- [同一 Turn 可以发送一次或多次 Responses API 请求](https://github.com/openai/codex/blob/6219b7c40fc9c702c0aef9964e72b492558f60e4/codex-rs/core/src/client.rs#L1-L19)；
- [工具调用产生后继续当前循环](https://github.com/openai/codex/blob/6219b7c40fc9c702c0aef9964e72b492558f60e4/codex-rs/core/src/stream_events_utils.rs#L287-L327)；
- [工具结果写回 Conversation History](https://github.com/openai/codex/blob/6219b7c40fc9c702c0aef9964e72b492558f60e4/codex-rs/core/src/session/turn.rs#L2118-L2142)。

## 理解检查

1. 为什么一个 Codex Turn 可以包含四次 Model Call？
2. 为什么 Tool Result 同时可以是一个 Item 和一次 Observation？
3. 为什么搜索到三个文件不等于已经理解整个退款功能？
4. 哪些状态由本地 Codex 保存，哪些只存在于模型推理 Runtime？
