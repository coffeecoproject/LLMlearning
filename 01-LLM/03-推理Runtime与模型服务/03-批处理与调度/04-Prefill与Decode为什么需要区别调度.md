---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-批处理与调度概览|批处理与调度概览]]"
previous: "[[03-静态Batch与Continuous-Batching有什么区别|静态Batch与Continuous Batching有什么区别]]"
next: "[[05-多请求调度完整案例|多请求调度完整案例]]"
tags: [llm, runtime, prefill, decode, scheduling]
---

# Prefill 与 Decode 为什么需要区别调度

> [!summary]
> Prefill 要读取一段已有输入，Decode 则反复推进新的输出位置；两者工作形态不同，混在一起时需要调度器避免长输入阻塞正在生成的请求。

## 两种工作形态

| 阶段 | 当前处理什么 | 用户感受 |
|---|---|---|
| Prefill | 已经给出的整段输入 Token | 还在等待第一个输出 |
| Decode | 在已有历史后反复预测新位置 | 正在逐步看到输出 |

一个 8,000 Token 的输入 Prefill，通常比一个 Decode 步骤包含更多待处理位置。如果它长时间独占设备，其他已经开始输出的请求可能出现明显停顿。

## 调度器在权衡什么

```text
尽快处理新请求的Prefill → 缩短首Token等待
持续安排现有请求Decode → 保持输出流畅
让设备尽量忙碌           → 提高总吞吐量
```

这三项目标有时互相冲突，因此不存在脱离请求分布和硬件条件的“永远最佳策略”。

## Chunked Prefill直白理解

一种实现会把很长的 Prefill 切成若干片段：

```text
长输入片段1 + 若干Decode请求
长输入片段2 + 若干Decode请求
……
```

这样长输入不必一次占据全部执行机会。它是 Runtime 的调度与执行策略，不是 Tokenizer 截断文本，也不是 Transformer 新增了一个层。

## 必须保留的模型边界

Prefill 可以并行处理已有输入中的多个位置，因为这些 Token 已经全部确定；Decode 的未来 Token 尚未确定，所以标准自回归生成仍需要一轮轮推进。区别调度利用了两阶段的工作形态，却没有改变因果注意力规则。

## 开放实现观察：vLLM

vLLM 的公开优化文档描述了 Chunked Prefill，并说明它可把 Prefill 与 Decode 请求共同纳入 Token 预算。公开实现证明这种策略存在，不表示所有服务都采用相同默认值。

来源：[vLLM Optimization and Tuning](https://docs.vllm.ai/en/latest/configuration/optimization/)，核对日期：2026-07-28。

## 理解检查

1. 为什么长 Prefill 会影响其他请求的输出连续性？
2. Chunked Prefill 与截断上下文有什么根本区别？

下一篇：[[05-多请求调度完整案例|多请求调度完整案例]]。
