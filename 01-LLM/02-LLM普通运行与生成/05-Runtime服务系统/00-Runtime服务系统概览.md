---
type: topic-index
module: 1
status: planned
audience: non-specialist
parent: "[[00-LLM普通运行与生成大纲|LLM普通运行与生成大纲]]"
previous: "[[00-单请求效率与资源概览|单请求效率与资源概览]]"
next: "[[00-普通运行与生成边界与复习概览|普通运行与生成边界与复习概览]]"
tags: [llm, inference, runtime, serving, batching]
---

# Runtime 服务系统概览

> [!summary]
> Runtime 是承载模型运行的软件系统：它加载权重、组织请求、管理设备与缓存，并把许多用户的生成步骤调度到有限计算资源上。

## 为什么必须单独成章

单请求机制回答：

```text
一个回答怎样逐 Token 产生？
```

Runtime 服务系统回答：

```text
许多长度不同、到达时间不同的请求
怎样共享 GPU、内存和模型副本？
```

后者不会替换 Prefill、KV Cache 和 Decode，而是在多个请求之上组织这些步骤。

## 计划阅读顺序

1. 模型、推理引擎、模型服务器、API 和客户端的边界；
2. Batch 与单请求的 Batch 维度；
3. Continuous Batching 怎样动态加入和移出请求；
4. Paged KV Cache / PagedAttention 解决什么内存管理问题；
5. 调度、并行、量化和部署形态的概览；
6. 本地运行、第三方托管与闭源 API 分别能观察什么。

## 当前证据边界

vLLM 官方文档公开 Continuous Batching、PagedAttention、Prefix Caching 和多类并行能力，可用于理解开放 Runtime。OpenAI 托管服务的内部批处理、KV Cache 布局与调度策略未公开时，应记录为未知，不能从 Codex CLI 或流式界面反推。

来源：[vLLM 官方文档](https://docs.vllm.ai/en/stable/)、[OpenAI Responses Streaming 官方参考](https://platform.openai.com/docs/api-reference/responses-streaming)，核对日期：2026-07-27。

> [!info] 当前进度
> 结构已建立，详细内容待补充。
