---
type: topic-index
module: 1
status: planned
audience: non-specialist
parent: "[[00-推理Runtime与模型服务大纲|推理Runtime与模型服务大纲]]"
previous: "[[01-Streaming为什么逐步显示|Streaming为什么逐步显示]]"
next: "[[01-LLM/04-LLM训练与对齐/00-概览/00-LLM训练与对齐大纲|LLM训练与对齐大纲]]"
tags: [llm, runtime, latency, throughput, boundaries]
---

# 性能指标与 Runtime 边界概览

> [!summary]
> Runtime 性能不能只看“模型每秒生成多少 Token”；还要区分首 Token 延迟、逐 Token 延迟、总延迟、单请求延迟、系统吞吐量和资源利用率。

## 计划内容

1. TTFT、TPOT、端到端延迟和 Tokens/s；
2. 单请求延迟与系统吞吐量为什么可能冲突；
3. 并发数、队列时间和服务等级；
4. 权重、激活、KV Cache 与工作区内存；
5. Runtime 优化、模型质量与生成策略的边界；
6. 客户端、Codex CLI 和托管服务端分别能证明什么。

完成本部分后，再进入 [[01-LLM/04-LLM训练与对齐/00-概览/00-LLM训练与对齐大纲|LLM 训练与对齐]]。
