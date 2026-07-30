---
type: concept-note
module: 2
status: complete
audience: non-specialist
phase: 运行阶段
parent: "[[00-推理与CoT框架速览概览|推理与 CoT 框架速览概览]]"
tags: [reasoning, cot, search, verification, agent, runtime, layers]
---

# 推理、CoT、搜索与验证分别在哪一层实现

> [!summary]
> 推理能力和 CoT 生成主要来自 LLM；模型 Runtime 负责高效执行生成；Agent 或推理控制器组织任务级搜索和验证循环；Verifier 或工具提供检查证据。

> [!note] 阅读层级
> 本篇属于“继续深入系统边界”。只看框架时，只需记住上面的四层分工即可；Codex、vLLM、部署方式和具体搜索机制都不是框架路线的必读内容。

## 为什么这个问题容易混乱

在实际产品里，我们经常只看到一个输入框：

```text
用户输入问题
→ 系统开始“思考”
→ 系统调用工具
→ 返回答案
```

界面把多个系统层隐藏在一起，因此很容易把下面这些事情都称为“模型推理”：

- Transformer 内部计算；
- 模型生成 CoT Token；
- Runtime 解码 Token；
- Agent 调用工具；
- 外部系统建立候选分支；
- 测试程序验证结果。

它们可以共同参与一次任务，但并不属于同一个逻辑层。

## 先区分逻辑层与物理服务器

“层”描述的是职责，不等于必须拥有一台独立服务器。

例如，Agent 和推理控制器可以是同一个程序：

```text
Agent Server
├── 上下文管理
├── 推理控制
├── 搜索与分支
├── 工具调用
└── 验证循环
```

也可以拆成：

```text
Agent Server
→ 推理控制服务
→ 模型 Runtime 服务
```

部署方式不同，不会改变它们各自承担的逻辑职责。

## 五个主要逻辑层

### 1. Agent、应用或推理控制器

主要负责：

- 整理用户目标和上下文；
- 选择模型与 Reasoning Effort；
- 决定是否建立多个候选；
- 保存搜索状态；
- 调用工具和 Verifier；
- 根据结果决定继续、回溯或停止。

小型系统通常由 Agent 直接承担推理控制器的职责，不必额外部署一个控制服务器。

### 2. 模型服务接口

它接收结构化请求，例如：

```text
模型名称
+ 指令与消息
+ 可用工具
+ 推理投入
+ 输出限制
```

然后把请求交给模型 Runtime。OpenAI Responses API、OpenAI 兼容 API 都属于这一层的常见接口形态。

接口层和 Runtime 可以由同一个服务提供，但概念上仍能区分：接口负责接收请求，Runtime 负责执行模型计算。

### 3. 模型 Runtime

例如 vLLM 一类 Runtime，主要负责：

- 加载模型 Weight 和 Tokenizer；
- 执行 Prefill 与 Decode；
- 管理 KV Cache；
- Batch 和请求调度；
- 根据采样规则选择 Token；
- 流式返回生成结果。

Runtime 可以高效执行三条候选路径，但通常不自行决定“为什么需要三条、保留哪条”。

### 4. LLM

LLM 的架构、Weight 和训练决定了它具备怎样的语言、知识与推理能力。

在一次调用中，LLM 会：

- 对当前上下文形成 Hidden State；
- 产生下一 Token 的 Logits；
- 逐步生成 Reasoning Token、CoT、工具调用或最终回答；
- 在后续模型调用中，根据追加到上下文的工具结果继续生成。

因此 CoT 内容由模型生成，但具体是否展示可以由服务接口或应用层决定。

### 5. Verifier 与工具

它们提供候选是否满足标准的证据，例如：

- 计算器重新计算结果；
- 编译器检查代码；
- 单元测试运行真实行为；
- 数据库返回订单状态；
- Verifier Model 给候选评分；
- 人类审阅开放性结果。

如果 Verifier 本身是模型，Runtime 负责运行它；上层控制器仍负责决定何时调用以及如何使用评分。

## 把概念映射到层级

| 概念 | 主要实现层 | 需要其他层怎样配合 |
|---|---|---|
| 基础推理能力 | LLM 的架构、Weight 与训练 | Runtime 执行模型 |
| 单次调用内部 Reasoning | LLM | 接口传入上下文和推理投入 |
| CoT Token | LLM 生成 | Runtime 解码，应用决定是否展示 |
| Reasoning Effort | Agent、客户端或配置选择 | 模型服务与模型执行 |
| Greedy、Top-p、Beam Search | 模型 Runtime | 应用提供参数 |
| 多候选生成 | 上层发起，Runtime 执行 | LLM 产生各条候选 |
| Self-Consistency | 推理控制器组织 | Runtime 生成候选，控制器投票 |
| Tree of Thoughts | 推理控制器维护搜索树 | LLM 扩展或评价节点 |
| Verification Loop | Agent 或推理控制器组织 | Verifier 提供证据，LLM根据结果修正 |
| Verifier Model | LLM + Runtime | 上层决定评价标准并使用评分 |

这里的“主要实现层”不是排他关系，而是指出哪一层对该能力负主要责任。

## 搜索为什么必须分成两类

### Token 级搜索与采样

```text
当前上下文
→ Runtime 根据 Logits 和解码规则选择下一个 Token
```

Greedy、Top-p Sampling、Beam Search 等通常属于模型 Runtime 的解码机制。

### 任务级搜索

```text
共同问题 P
├── 候选 A
├── 候选 B
└── 候选 C
     ↓
评价、淘汰、扩展或回溯
```

Self-Consistency、Tree of Thoughts、代码库搜索和多方案验证通常由 Agent 或推理控制器组织。Runtime 只负责高效执行其中的模型请求。

详见 [[00-采样搜索与验证概览|采样、搜索与验证]]。

## Verification 为什么也分成组织与执行

一次完整验证包含：

```text
确定验收标准
→ 选择 Verifier
→ 执行检查
→ 读取证据
→ 决定接受、修正或继续
```

其中：

- Agent 或推理控制器组织整个循环；
- 测试、计算器、数据库或 Verifier Model 执行具体检查；
- LLM 可以根据失败证据重新生成；
- Runtime 只在检查涉及模型调用时负责模型执行。

必须同时记住：

```text
具备 Verification Loop
≠ 每次都能发现全部验证条件
```

如果模型遗漏了并发测试，即使它真实运行了已有单元测试，验证范围仍可能不完整。

## 真实系统观察：Codex CLI

OpenAI 公开资料将 Codex CLI 的核心描述为 Agent Loop 或 Agent Harness。它在本地组织用户、模型和工具之间的循环：

```text
用户输入
→ Codex CLI 构建指令、工具、权限与环境上下文
→ 调用 Responses API
→ 模型返回工具调用或最终消息
→ Codex CLI 执行工具
→ 工具结果追加到下一次模型请求
→ 重复直到模型返回最终消息
```

因此，Codex CLI 已经同时承担：

- 本地 Agent；
- 上下文构建；
- 工具执行器；
- 基础推理控制循环；
- Verification 的执行与反馈循环。

但具体“应该读取哪些文件、运行哪些测试、是否还要继续检查”，通常仍由模型根据指令和当前证据判断。

所以更准确的结论是：

```text
Codex 具备完整可运行的 Verification Loop
≠ Codex 每次自动找到全部验证范围
```

Codex CLI 还可以选择模型和 Reasoning Effort；真正的单次模型 Reasoning 在模型服务和 Codex 模型中执行。OpenAI 没有公开远端模型 Runtime 的全部内部实现，因此不能把它直接等同于 vLLM。

## 两种常见部署方式

### Agent 直接包含推理控制

```text
用户
→ Agent（含推理控制、工具和验证循环）
→ 模型服务 / Runtime
→ LLM
```

这是多数个人项目和普通 Agent 的合理起点。

### 独立推理控制服务

```text
用户
→ Agent / 业务服务
→ 推理控制服务
→ 模型服务 / Runtime
→ LLM
```

只有多个 Agent 需要共享搜索策略、集中控制预算或大规模并行时，才更有必要拆分。

## 常见误解

> [!warning] 误解：vLLM 让模型获得了推理能力
> vLLM 负责模型 Inference；模型的 Reasoning 能力主要来自架构、Weight 和训练。

> [!warning] 误解：Agent 执行了一次测试，就已经完成全部验证
> 测试结果可以是真实证据，但是否覆盖了所有验收条件仍需判断。

> [!warning] 误解：服务器一定是一个单独逻辑层
> Agent、控制器、接口和 Runtime 可以部署在同一台机器，也可以拆开。应先按职责理解，再看物理部署。

> [!warning] 误解：一次高 Reasoning Effort 会自动建立搜索树
> Reasoning Effort 主要控制每条调用的推理投入；任务级分支仍需要上层控制。

## 主要来源

> [!source]
> - [OpenAI：Unrolling the Codex agent loop](https://openai.com/index/unrolling-the-codex-agent-loop/)，公开说明 Codex CLI 如何构建输入、调用 Responses API、执行工具并把结果加入下一轮请求。
> - [Codex 配置参考](https://learn.chatgpt.com/docs/config-file/config-reference)，说明 Codex 可配置模型、Reasoning Effort、工具、权限与 Sandbox 等运行条件。
> - [vLLM 官方仓库](https://github.com/vllm-project/vllm)，将 vLLM 定位为 LLM Inference and Serving Engine。
> - 核对日期：2026-07-29。Codex 和模型服务接口属于版本敏感信息。

## 理解检查

1. 为什么逻辑层不一定对应独立服务器？
2. CoT 内容由谁生成，又由谁决定是否展示？
3. Token 级搜索和任务级搜索分别主要在哪一层？
4. Codex 为什么具备 Verification Loop，却仍可能遗漏验证范围？
5. vLLM 与 LLM 的 Reasoning 能力有什么区别？
