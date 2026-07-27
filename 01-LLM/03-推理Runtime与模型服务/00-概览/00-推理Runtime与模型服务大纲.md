---
type: section-outline
module: 1
status: active
audience: non-specialist
parent: "[[01-LLM/00-概览/00-LLM 模块大纲|LLM 模块大纲]]"
previous: "[[00-单请求推理与生成边界与复习概览|单请求推理与生成边界与复习概览]]"
next: "[[00-推理Runtime框架速览概览|推理Runtime框架速览概览]]"
tags: [llm, inference, runtime, serving, outline]
---

# 推理 Runtime 与模型服务大纲

> [!summary]
> Runtime 是在模型结构之外承载推理的软件系统：它加载权重、接收请求、组织模型前向计算、管理缓存和设备，并把结果通过接口交给应用。

## 为什么从单请求生成中独立出来

[[00-单请求推理与生成大纲|单请求推理与生成]]回答：

```text
一个回答为什么必须逐 Token 产生？
```

Runtime 回答：

```text
许多到达时间、输入长度和输出长度不同的请求
怎样共享模型、GPU、内存和网络接口？
```

两者连接，但研究对象不同：

| 单请求生成机制 | Runtime 服务系统 |
|---|---|
| Prefill 为什么需要处理上下文 | Prefill 请求怎样排队和调度 |
| KV Cache 为什么可以复用 | KV Cache 怎样分页、分配和回收 |
| Decode 为什么逐 Token 进行 | 多个 Decode 步骤怎样动态组成 Batch |
| Tokenizer Decode 怎样恢复文字 | 文字怎样封装成 Streaming/API 事件 |

## Runtime 在完整系统中的位置

```text
应用或 Agent
→ API / 模型服务器
→ 推理 Runtime
→ 已加载的模型权重
→ CPU / GPU / 其他加速设备
```

Runtime 不是模型的 Attention 或 FFN，也不是 Agent 的规划器。它是让固定参数模型可以被高效调用的执行与服务层。

## 六部分结构

1. [[00-推理Runtime框架速览概览|框架速览]]：一页看清模型、Runtime、服务端和客户端；
2. [[00-模型加载与请求边界概览|模型加载与请求边界]]：权重、Tokenizer、配置和一次请求怎样进入引擎；
3. [[00-批处理与调度概览|批处理与调度]]：Batch、Continuous Batching 与请求生命周期；
4. [[00-KV-Cache工程管理概览|KV Cache 工程管理]]：分页、内存块、Prefix Cache 和回收；
5. [[00-接口流式传输与部署概览|接口、流式传输与部署]]：Streaming、API、本地与托管服务；
6. [[00-性能指标与Runtime边界概览|性能指标与边界]]：吞吐量、延迟、资源和证据边界。

## 本部分不重复讲什么

- 不重新解释 Attention 为什么产生 Q、K、V；
- 不重新推导 Prefill 和 Decode 的单请求机制；
- 不把 Chat Template、Temperature、Top-p 当作 Runtime 的核心结构；
- 不展开 Agent 规划、工具调用和任务循环；
- 不根据客户端源码反推闭源服务端内部架构。

## 真实实现观察

- Hugging Face Transformers 可以在单进程中加载 Tokenizer、模型并执行 `generate`，适合观察基础推理软件接口。
- vLLM 明确提供 Continuous Batching、PagedAttention、Prefix Caching、Streaming、并行和 API Server，适合观察服务型 Runtime。
- OpenAI Responses API 公开输入、输出和 Streaming 事件协议，但托管服务内部的调度、缓存布局和模型部署结构若未公开，仍属于未知信息。

来源：[Hugging Face Transformers Generation](https://huggingface.co/docs/transformers/main_classes/text_generation)、[vLLM 官方文档](https://docs.vllm.ai/en/stable/)、[OpenAI Responses Streaming 官方参考](https://platform.openai.com/docs/api-reference/responses-streaming)，核对日期：2026-07-27。

## 完成本部分后应能回答

> 模型权重本身不会接收网络请求，那么一个模型服务怎样把许多用户请求转化为实际的 Prefill 和 Decode 计算，并管理它们的资源与输出？
