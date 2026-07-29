---
type: optional-concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-执行优化并行与部署概览|执行优化并行与部署概览]]"
previous: "[[08-Kernel融合编译与推测解码在优化什么|Kernel融合编译与推测解码在优化什么]]"
next: "[[00-性能指标与测量边界概览|性能指标与测量边界概览]]"
tags: [llm, runtime, parallelism, multi-gpu, optional]
---

# 多 GPU 并行方式分别在拆什么

> [!optional] 工程选读
> 目标是看懂名称，不要求学习通信公式或部署配置。

> [!summary]
> 不同并行方式拆分的对象不同：有的拆一层计算，有的拆模型层，有的复制整份模型处理不同请求，还有的专门拆 MoE 专家或长上下文。

| 名称 | 主要拆什么 | 直白理解 |
|---|---|---|
| Tensor Parallel（TP） | 同一层内部的张量计算 | 一层太大，多卡共同算这一层 |
| Pipeline Parallel（PP） | 不同模型层 | 前几层在一组设备，后几层在另一组 |
| Data Parallel（DP） | 请求或数据 | 多份模型副本分别处理不同请求 |
| Expert Parallel（EP） | MoE 中的专家 | 不同专家分布在不同设备 |
| Context Parallel（CP） | 长序列位置或注意力工作 | 多设备分担很长上下文的处理 |

## 为什么没有一种方式永远最好

拆分越细，越可能需要频繁通信；复制越多，权重占用也越多。模型结构、单机互联、跨机网络、请求长度和流量都会影响选择。

现实部署还可以组合，例如 TP=4 的一个模型副本，再用 DP 部署两个副本。看到“用了 8 张 GPU”仍不足以推断具体结构。

## Prefill与Decode分离部署

还有一种服务组织把 Prefill 与 Decode 放在不同实例，并传递 KV Cache，以便分别调节首 Token 延迟和后续 Token 延迟。vLLM 将其公开为实验性功能，并明确指出这种分离本身不保证提高吞吐量，因此这里只作概念观察。

来源：[vLLM Parallelism and Scaling](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/)、[vLLM Context Parallel](https://docs.vllm.ai/en/latest/serving/context_parallel_deployment/)、[vLLM Disaggregated Prefill](https://docs.vllm.ai/en/latest/features/disagg_prefill/)，核对日期：2026-07-28。

## 理解检查

1. TP 与 DP 最大的区别是什么？
2. 为什么 Prefill/Decode 分离不能直接等于吞吐量提升？

下一专题：[[00-性能指标与测量边界概览|性能指标与测量边界]]。
