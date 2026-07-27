---
type: section-outline
module: 1
status: active
audience: non-specialist
parent: "[[01-LLM/00-概览/00-LLM 模块大纲|LLM 模块大纲]]"
next: "[[00-普通运行与生成框架速览概览|普通运行与生成框架速览]]"
tags: [llm, inference, generation, runtime, outline]
---

# LLM 普通运行与生成大纲

> [!summary]
> 本部分研究一个**已经训练完成、参数保持固定**的 LLM，怎样把一次用户输入变成逐 Token 输出；先讲单请求的模型主线，最后再单独讲多请求 Runtime 服务系统。

## 先看完整位置

```text
用户消息
→ 输入协议与 Tokenizer
→ Prefill：处理现有上下文
→ 得到下一 Token 的 Logits
→ 选择一个 Token
→ Decode：利用 KV Cache 逐步续写
→ 满足停止条件
→ Tokenizer Decode 与流式显示
```

这条链同时经过几个不同层次：

```text
应用输入层：Messages、Chat Template
模型输入层：input_ids、Mask、Position 信息
模型计算层：Embedding、Transformer、Output Layer
生成控制层：Logits 处理、选择策略、停止条件
服务系统层：Batch、调度、KV Cache 内存管理、并行部署
```

这些层互相连接，但不能全部叫作“模型内部结构”。

## 六部分结构

1. [[00-普通运行与生成框架速览概览|框架速览]]：一页建立完整心智模型；
2. [[00-单请求运行与生成概览|单请求运行与生成]]：从输入准备到逐 Token 输出的核心主线；
3. [[00-上下文与多轮交互概览|上下文与多轮交互]]：上下文窗口、截断与对话历史；
4. [[00-单请求效率与资源概览|单请求效率与资源]]：输入长度、输出长度、延迟和 KV Cache 内存；
5. [[00-Runtime服务系统概览|Runtime 服务系统]]：多个请求怎样被调度和批处理；
6. [[00-普通运行与生成边界与复习概览|边界与复习]]：重新区分训练、模型、Runtime、API 与 Agent。

## 按学习目标选择路线

### 只想理解框架

阅读 [[00-普通运行与生成框架速览概览|普通运行与生成框架速览]]。能够复述“输入准备 → Prefill → Token 选择 → 增量 Decode → 停止与显示”后，可以先停在这里。

### 想理解一次回答怎样真正产生

继续阅读 [[00-单请求运行与生成概览|单请求运行与生成]]。这是本部分的核心必读内容。

### 想理解对话、性能和部署

再按顺序进入上下文、单请求效率和 Runtime 服务系统。它们建立在单请求主线之上，不应提前混入模型基础结构。

## 运行阶段的关键边界

> [!info] 运行阶段
> 模型参数、Embedding Matrix、Attention 投影和 FFN 权重通常保持不变；当前输入的 Hidden States、KV Cache、已生成 Token 和停止状态会随请求变化。

本部分不展开：

- Loss、反向传播、优化器和参数更新；
- CoT 为什么能改善复杂任务表现；
- Agent 怎样规划、调用工具和循环执行；
- RAG 怎样检索外部知识。

## 真实实现观察

- `Qwen/Qwen3-8B` 官方示例把 `messages` 交给 `apply_chat_template`，得到模型输入后再调用 `generate`，说明“消息协议”和“模型前向计算”是相邻但不同的环节。
- Hugging Face Transformers 官方文档把 KV Cache 描述为对以往 Attention 的 K、V 计算结果的复用，并明确它用于推理而非普通训练。
- vLLM 官方文档把 Continuous Batching、PagedAttention 和调度列为推理服务能力，说明它们属于 Runtime 服务层，不是某一个 Transformer Block 的结构。
- OpenAI 托管模型可以从官方接口观察消息、流式事件等外部行为；未公开的内部 Prefill、KV Cache 组织和调度实现仍然是未知信息。`gpt-oss` 开放权重模型则可以用于检查公开的本地运行路径。

来源：[Qwen3-8B 官方模型页](https://huggingface.co/Qwen/Qwen3-8B)、[Hugging Face Caching 官方文档](https://huggingface.co/docs/transformers/cache_explanation)、[vLLM 官方文档](https://docs.vllm.ai/en/stable/)、[OpenAI Responses Streaming 官方参考](https://platform.openai.com/docs/api-reference/responses-streaming)，核对日期：2026-07-27。

## 完成本部分后应能回答

> 当用户发送一条消息后，应用、Tokenizer、固定参数模型、生成控制器和 Runtime 分别做了什么？为什么输出必须一个 Token 接一个 Token 地形成？
