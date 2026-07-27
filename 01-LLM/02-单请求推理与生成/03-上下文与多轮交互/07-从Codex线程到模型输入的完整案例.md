---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-上下文与多轮交互概览|上下文与多轮交互概览]]"
previous: "[[06-一次多轮对话怎样受到窗口限制|一次多轮对话怎样受到窗口限制]]"
next: "[[00-单请求效率与资源概览|单请求效率与资源概览]]"
tags: [llm, inference, context, multi-turn, codex, runtime-boundary, case-study]
---

# 从 Codex 线程到模型输入的完整案例

> [!summary]
> Codex 不会把聊天界面直接交给 LLM：它先接收本轮输入，把输入写入线程记录和模型可见历史，再与当前指令、环境及工具组合成一次模型请求；请求到达模型服务后，文本才真正经过 Tokenizer。

> [!phase] 运行阶段·跨层案例
> 本篇以开源 Codex CLI `rust-v0.145.0` 为观察对象，重点解释“多轮 Thread 怎样变成一次 LLM 输入”。Codex 的对话管理和工具循环属于应用或 Agent Runtime；Tokenizer、Embedding 与 Transformer 计算属于模型侧。两者会在一次真实请求中相接，但不是同一层。

## 先看这次真实问题

本篇沿用当前学习线程中的一句真实用户输入：

```text
那基于源码 比如我们当前线程 那这段对话是怎么进入的 完整示例出来
```

用户看到的起点和终点是：

```text
在 Codex Desktop 中发送一句话
→ 稍后看到 Codex 的回答
```

中间实际跨过了两套系统：

```text
Codex 应用与 Agent Runtime
负责：保存对话、选择上下文、组织工具、调用模型

LLM 与模型服务
负责：Tokenize、Prefill、逐 Token 生成
```

如果不先分开这两套系统，就容易误以为“用户文字一提交，立刻进入 Embedding”。

## 四个对象不要混在一起

| 对象 | 直白理解 | 是否全部交给模型 |
|---|---|---|
| Thread | 这场长期对话的档案 | 不一定 |
| Turn | 用户发起的一轮任务 | 本轮输入会成为候选内容 |
| 模型可见历史 | Codex 为当前采样整理出的有效内容 | 会 |
| KV Cache / Prompt Cache | 对已处理前缀的计算或服务优化 | 不是一条聊天消息 |

最重要的关系是：

```text
Thread 中存在某条消息
不等于
这条消息仍逐字存在于本次模型可见历史
```

## 完整主线

```text
普通 Unicode 字符串
→ turn/start 请求
→ UserInput::Text
→ Op::UserInput
→ role = user 的 ResponseItem
→ 写入 Thread 持久记录
→ 写入 ContextManager
→ 加入当前指令、环境、历史和工具定义
→ 构造 Responses API 请求
→ 模型服务执行 Tokenizer
→ Token ID 与 Embedding
→ Transformer 计算
→ 返回文字或工具调用
→ 若调用工具，结果进入历史后再次请求模型
→ 返回最终回答
```

下面逐步拆开。

## 第一步：Desktop 提交一个新 Turn

如果上一轮已经结束，用户再次发送消息，App Server 协议使用 `turn/start` 开始新 Turn。下面是根据公开协议写出的**简化示例**，线程 ID 为教学占位符：

```json
{
  "method": "turn/start",
  "params": {
    "threadId": "<当前线程 ID>",
    "input": [
      {
        "type": "text",
        "text": "那基于源码 比如我们当前线程 那这段对话是怎么进入的 完整示例出来",
        "textElements": []
      }
    ]
  }
}
```

此时最核心的数据仍然是普通字符串：

```text
"那基于源码……"
```

它还没有被切成 Token，也没有 Token ID，更没有进入 Embedding。

`turn/start` 还可以携带本轮覆盖设置，例如模型、工作目录、权限、沙箱策略和推理强度。若没有提供，Codex 可以继续使用 Thread 中已经保存的设置。

> [!note] 用户在运行中继续发消息
> 如果当前 Turn 尚未结束，新输入可能通过 `turn/steer` 进入待处理队列，在下一次合适的模型请求前并入历史。这与“上一轮结束后再开启一个 Turn”不同。

## 第二步：App Server 把输入交给 Codex Core

App Server 会找到对应 Thread，检查输入限制，把协议对象转换为 Core 对象，然后创建 `Op::UserInput`。概念上类似：

```text
Op::UserInput
├── items
│   └── UserInput::Text
│       └── text = "那基于源码……"
├── additional_context
├── thread_settings
└── final_output_json_schema
```

这里的 `Op` 可以理解成“交给 Codex 核心处理的一项操作”。它不是模型 Token，也不是 Transformer 的输入张量。

Codex 随后为这次提交生成 Turn ID，建立本轮运行上下文，并启动普通 Agent Turn。

## 第三步：字符串变成结构化用户消息

Codex Core 会把文本输入转换成模型 API 使用的消息对象。简化后是：

```json
{
  "type": "message",
  "role": "user",
  "content": [
    {
      "type": "input_text",
      "text": "那基于源码 比如我们当前线程 那这段对话是怎么进入的 完整示例出来"
    }
  ]
}
```

此时发生的是：

```text
一段没有角色的输入文字
→ 一条明确标记为 user 的结构化消息
```

但它仍然是文本。`role: user`、`type: input_text` 这些结构以后还需要按照目标模型的输入协议进行编码，才会成为模型真正计算的 Token 序列。

## 第四步：同一条消息承担两种作用

Codex 会把这条用户消息记录到两个有关联但不同的地方。

### A. Thread 持久记录

它用于保留对话档案，例如：

- 在界面显示历史；
- 恢复或 Fork Thread；
- 将工具调用和输出归入对应 Turn；
- 在重新加载时重建当前状态。

### B. ContextManager 模型可见历史

它用于构造下一次模型请求。`ContextManager` 会保存当前有效的用户消息、助手消息、工具调用和工具结果，并在发出请求前进行规范化。

所以同一句话会同时成为：

```text
供产品保存和显示的对话记录
+
供本轮模型推理使用的上下文内容
```

但两者以后可能分离。例如 Thread 仍保留旧消息，而模型可见历史已经用摘要替换了部分旧细节。

## 第五步：Codex 组织“本轮真正要看的内容”

模型请求不只有当前一句话。Codex 还会准备：

- 模型基础指令；
- 当前工作目录与环境信息；
- `AGENTS.md` 等项目约束；
- 权限、沙箱和协作模式；
- Skills、Plugins 及本轮可用工具；
- 当前仍有效的历史消息；
- 当前用户问题。

用当前学习线程表示，**逻辑结构**可能类似：

```text
模型基础指令

Developer Context
├── 当前工作区：/Users/liushan/Docs/LLMLearning
├── 当前权限与运行环境
├── AGENTS.md 中的学习笔记规范
└── 当前可使用的工具说明

有效历史
├── 用户：询问 KV Cache 是否进入上下文
├── 助手：解释 Thread、模型上下文与 Cache 的区别
├── 用户：要求根据 Codex 源码查看上下文处理
├── 助手：给出 ContextManager 和 Compaction 结论
└── ……其他仍被保留或已经被摘要的信息

当前消息
└── 用户：那基于源码……完整示例出来
```

这只是帮助理解的结构化重建，不是当前私有请求的原始抓包。公开源码能确认组装机制，但不能让我们从仓库中读取某个真实运行实例的全部私有 Prompt。

## 第六步：ContextManager 在发送前整理历史

Codex 不会无条件把所有记录原样发送。发送前至少会处理这些问题：

1. 工具调用是否有对应的工具结果；
2. 是否存在没有来源的孤立工具结果；
3. 工具返回内容是否太长，需要截断；
4. 当前模型是否支持历史中的图片或音频；
5. 历史是否已经到达上下文压缩条件。

如果 Thread 很长并触发了 Compaction，新的模型可见历史可能变为：

```text
重新注入的当前指令与环境
+
保留下来的近期用户消息
+
对旧对话的压缩摘要
+
压缩后产生的新消息与工具结果
```

因此：

```text
在界面中还能向上翻到旧回答
≠
旧回答仍然逐字进入本轮 Prefill
```

摘要保留的是被压缩系统判断为重要的状态，不保证保留每一个原句和细节。

## 第七步：组成逻辑上的模型请求

`ContextManager` 整理完成后，Codex 构造的 `Prompt` 包含：

```text
Prompt
├── base_instructions：模型基础指令
├── input：整理后的模型可见历史
├── tools：本轮允许模型调用的工具定义
├── parallel_tool_calls：是否允许并行工具调用
└── output_schema：可选的输出格式要求
```

对应到 Responses API，请求可以简化理解为：

```json
{
  "model": "<当前模型>",
  "instructions": "<模型基础指令>",
  "input": [
    {"role": "developer", "content": "<当前环境与约束>"},
    {"role": "user", "content": "<有效的先前消息或摘要>"},
    {"role": "assistant", "content": "<仍保留的助手输出>"},
    {
      "role": "user",
      "content": "那基于源码 比如我们当前线程 那这段对话是怎么进入的 完整示例出来"
    }
  ],
  "tools": ["<当前模型可见的工具定义>"],
  "tool_choice": "auto",
  "stream": true,
  "prompt_cache_key": "<会话缓存标识>"
}
```

> [!warning] 这是结构示例
> 字段关系来自源码，但其中的 ID、指令内容、历史选取和工具列表使用了占位符。它不能被当作当前会话的真实网络抓包。

## 第八步：到模型服务后才进入 Tokenizer

在 Codex 侧，用户内容主要仍以字符串和消息对象存在。请求到达模型服务后，服务才使用与目标模型匹配的 Tokenizer，把消息协议和文本变成 Token ID。

```text
Messages 与指令
→ 模型配套的输入协议
→ Tokenizer
→ Token ID 序列
→ Token Embedding
→ Position
→ Transformer Blocks
```

为了理解，可以把变化想象为：

```text
"那基于源码……"
→ ["那", "基于", "源码", "……"]
→ [示例 ID 1, 示例 ID 2, 示例 ID 3, ……]
```

这里的切分和 ID 都是教学化占位符，不代表当前 OpenAI 模型的真实 Tokenizer 结果。Codex CLI 可以估算上下文长度，但源码明确把本地估算描述为粗略估计，而不是 Tokenizer 级的精确计数。

## 第九步：为什么这次回答会出现多次模型请求

用户要求“基于源码”，模型可能先返回工具调用，而不是直接给最终答案：

```text
第一次模型请求
→ 模型判断需要检查源码
→ 返回工具调用

Codex 执行工具
→ 得到源码片段或搜索结果
→ 把工具调用与结果写入模型可见历史

第二次模型请求
→ 模型看到原问题 + 工具调用 + 工具结果
→ 决定是否继续调用工具或给出最终回答
```

模型可见历史会因此增长为：

```text
……原有上下文
User：那基于源码……
Assistant：调用源码检查工具
Tool：返回相关源码内容
Assistant：根据源码形成最终回答
```

一次用户 Turn 因而可能包含多次模型采样。Agent Loop 管理“什么时候调用工具和再次请求”；LLM 每次只处理当次实际提交的上下文。

## 传输优化不等于改变逻辑上下文

Codex 源码还包含 `prompt_cache_key` 和 `previous_response_id` 等机制。在满足条件时，同一 Turn 后续请求可以复用前一次响应，只发送新增部分。

但应区分：

```text
逻辑上：模型需要延续前面的有效上下文

传输上：客户端可能不必重复发送全部相同字节

计算上：模型服务可能复用部分前缀计算
```

这不能证明 Codex CLI 自己保存和操作 Transformer 的 KV Cache。服务器内部怎样保存、分页和淘汰 KV，公开 CLI 源码并不可见。

## 哪些事实能从源码确认

### 已确认

- App Server 使用 Thread、Turn 和 Item 组织对话；
- `turn/start` 的输入可以包含文本、图片、音频、Skill 或 Mention；
- 文本会被转换成 `role: user` 的结构化消息；
- `ContextManager` 管理当前模型可见历史；
- Prompt 将历史、基础指令和工具定义组合起来；
- 工具结果会进入历史并触发后续模型请求；
- 上下文过长时存在本地或远程 Compaction 路线；
- WebSocket 请求在符合条件时可以使用增量输入和 `previous_response_id`。

### 根据源码做出的合理推断

- 上一轮已经完成后发送的新消息，通常走新的 `turn/start`；
- 当前 Thread 很长时，某些较早细节可能已经只通过摘要继续存在；
- 缓存命中可以减少传输或计算，但不会把未选入上下文的新信息自动交给模型。

### 公开源码无法确认

- 当前私有请求最终发送的全部隐藏指令原文；
- OpenAI 模型使用的完整私有 Tokenizer 配置与真实 Token ID；
- 模型服务器内部 KV Cache 的具体布局和淘汰策略；
- 远程 Compaction 服务内部如何决定摘要的每个细节。

## 常见误解

1. **“用户文字直接进入 Embedding。”** 在真实聊天产品中，通常先经过消息协议、上下文组织和 Tokenizer。
2. **“Thread 就是 Context Window。”** Thread 是长期记录；Context Window 限制本次模型实际可见内容。
3. **“界面能看到，所以模型也能看到。”** 界面历史可能比模型可见历史更完整。
4. **“工具自己知道前面的对话。”** 工具通常只接收 Agent 明确传给它的参数；工具结果再由 Agent 写回历史。
5. **“previous_response_id 就是 KV Cache。”** 它是请求延续机制；服务器是否以及怎样复用 KV 是另一层实现。
6. **“Codex 组织了 Messages，所以 Codex 就是 LLM。”** Codex 是使用 LLM 的 Agent 系统，LLM 是其中负责模型计算的核心组件。

## 用一句话重新复述

```text
Codex 把长期 Thread 整理成有限的模型可见请求；模型服务再把请求 Tokenize，并用 LLM 进行本轮计算。
```

## 理解检查

1. 用户刚输入的中文句子，在 `UserInput::Text` 阶段是不是 Token？
2. 为什么 Thread 中保留一条消息，不保证模型本轮逐字看到它？
3. `ContextManager` 和模型上下文窗口分别负责什么？
4. 工具执行完以后，模型为什么还需要再次请求？
5. `previous_response_id`、Prompt Cache 与 KV Cache 为什么不能当成同一个概念？
6. 从哪一步开始，当前问题才真正进入已经学习过的 Tokenizer → Embedding → Transformer 主线？

## 开源实现来源

观察版本：OpenAI Codex CLI `rust-v0.145.0`；核对日期：2026-07-27。

- [Codex App Server 官方说明](https://learn.chatgpt.com/docs/app-server)
- [`TurnStartParams` 与输入类型](https://github.com/openai/codex/blob/rust-v0.145.0/codex-rs/app-server-protocol/src/protocol/v2/turn.rs)
- [App Server 的 Turn 处理](https://github.com/openai/codex/blob/rust-v0.145.0/codex-rs/app-server/src/request_processors/turn_processor.rs)
- [用户输入转换为模型消息](https://github.com/openai/codex/blob/rust-v0.145.0/codex-rs/protocol/src/models.rs)
- [`ContextManager` 模型可见历史](https://github.com/openai/codex/blob/rust-v0.145.0/codex-rs/core/src/context_manager/history.rs)
- [Agent Turn 与工具循环](https://github.com/openai/codex/blob/rust-v0.145.0/codex-rs/core/src/session/turn.rs)
- [上下文压缩](https://github.com/openai/codex/blob/rust-v0.145.0/codex-rs/core/src/compact.rs)
- [Responses API 请求与增量传输](https://github.com/openai/codex/blob/rust-v0.145.0/codex-rs/core/src/client.rs)

下一专题：[[00-单请求效率与资源概览|单请求效率与资源]]。
