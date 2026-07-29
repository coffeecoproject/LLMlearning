---
type: section-outline
module: 1
status: complete
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

## 八部分结构

1. [[00-推理Runtime框架速览概览|框架速览]]：一页看清模型、Runtime、服务端和客户端；
2. [[00-模型加载与请求生命周期概览|模型加载与请求生命周期]]：模型怎样从文件变成可执行服务，一个请求又怎样开始和结束；
3. [[00-批处理与调度概览|批处理与调度]]：Batch、Continuous Batching 与多个请求怎样共享执行机会；
4. [[00-KV-Cache工程管理概览|KV Cache 工程管理]]：分页、内存块、Prefix Cache 和回收；
5. [[00-API与流式传输概览|API 与流式传输]]：客户端、模型服务器、响应事件与网络边界；
6. [[00-执行优化并行与部署概览|执行优化、并行与部署]]：设备、精度、执行组织和多设备服务；
7. [[00-性能指标与测量边界概览|性能指标与测量边界]]：吞吐量、延迟、资源与测量条件；
8. [[00-推理Runtime边界与复习概览|边界与复习]]：完整串联系统职责、状态和可观察证据。

## 学习尺度

Runtime 涉及大量工程名词，但本部分仍然沿用前两部分的学习方式：

```text
先看系统为什么需要这一层
→ 再看对象和职责
→ 再看一条真实运行过程
→ 最后才接触实现变体和可选计算
```

必修主线回答：

- 这个组件为什么存在；
- 它接收什么、产生什么；
- 它与前后环节怎样连接；
- 没有它会出现什么问题；
- 它改变速度、容量、状态还是模型能力。

以下内容不作为理解 Runtime 的前提：显存地址、CUDA 编程、并行通信公式、调度算法证明和具体云平台配置。需要时只使用简单数字说明因果关系。

## 本部分不重复讲什么

- 不重新解释 Attention 为什么产生 Q、K、V；
- 不重新推导 Prefill 和 Decode 的单请求机制；
- 不把 Chat Template、Temperature、Top-p 当作 Runtime 的核心结构；
- 不展开 Agent 规划、工具调用和任务循环；
- 不把部署命令、框架参数或硬件公式当成概念主线；
- 不根据客户端源码反推闭源服务端内部架构。

## 真实实现观察

- Hugging Face Transformers 可以在单进程中加载 Tokenizer、模型并执行 `generate`，适合观察基础推理软件接口。
- vLLM 明确提供 Continuous Batching、PagedAttention、Prefix Caching、Streaming、并行和 API Server，适合观察服务型 Runtime。
- OpenAI Responses API 公开输入、输出和 Streaming 事件协议，但托管服务内部的调度、缓存布局和模型部署结构若未公开，仍属于未知信息。

来源：[Hugging Face Transformers Generation](https://huggingface.co/docs/transformers/main_classes/text_generation)、[vLLM 官方文档](https://docs.vllm.ai/en/stable/)、[OpenAI Responses Streaming 官方参考](https://platform.openai.com/docs/api-reference/responses-streaming)，核对日期：2026-07-28。

> [!note] 阅读深度
> 1—5 与第 8 部分构成用户理解主线。第 6 部分只必读设备、部署与优化取舍；Kernel、并行变体和 Prefill/Decode 分离均已标为选读。第 7 部分只要求会读指标和测试条件，不要求硬件容量计算。

## 完成本部分后应能回答

> 模型权重本身不会接收网络请求，那么一个模型服务怎样把许多用户请求转化为实际的 Prefill 和 Decode 计算，并管理它们的资源与输出？

本部分内容已补齐。完成 [[00-推理Runtime边界与复习概览|边界与复习]] 后，可进入 [[01-LLM/04-LLM训练与对齐/00-概览/00-LLM训练与对齐大纲|LLM训练与对齐]]。
