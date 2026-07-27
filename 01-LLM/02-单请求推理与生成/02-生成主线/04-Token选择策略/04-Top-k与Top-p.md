---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Token选择策略概览|Token选择策略概览]]"
previous: "[[03-Temperature怎样影响分布|Temperature怎样影响分布]]"
next: "[[00-停止与文本恢复概览|停止与文本恢复概览]]"
tags: [llm, generation, top-k, top-p, nucleus-sampling]
---

# Top-k 与 Top-p

> [!summary]
> Top-k 固定保留概率最高的 k 个候选；Top-p 动态保留累计概率达到阈值 p 的最小候选集合。

## Top-k：固定候选数量

候选概率：

```text
A 0.45
B 0.30
C 0.15
D 0.07
E 0.03
```

如果 `top_k = 3`，只保留 A、B、C，再把它们重新归一化后抽样。无论分布很集中还是很分散，候选数量都固定为 3。

## Top-p：固定累计概率范围

如果 `top_p = 0.80`：

```text
A：累计 0.45
B：累计 0.75
C：累计 0.90  ← 首次达到或超过 0.80
```

因此保留 A、B、C。Top-p 的候选数量会随每一步分布变化：模型很确定时集合可能很小，模型不确定时集合可能更大。

## 两者可以一起使用吗

可以，但具体组合顺序和边界行为由生成库实现。一起使用时，候选集合可能同时受数量上限和累计概率约束。学习概念时先分别理解，不需要把所有参数都同时打开。

## 与 Temperature 的区别

| 参数 | 主要改变什么 |
|---|---|
| Temperature | 调整候选概率的相对尖锐程度 |
| Top-k | 限制保留的候选数量 |
| Top-p | 限制保留的累计概率范围 |

它们通常发生在最终 Sampling 之前。

## 开放模型观察

Hugging Face `GenerationConfig` 官方文档把 `temperature`、`top_k` 和 `top_p` 作为生成配置项。`Qwen/Qwen3-8B` 官方模型页给出推荐生成参数，这说明参数常由模型发布者针对该版本提供建议；它们不是所有 LLM 的统一最佳值。

来源：[Hugging Face Generation 官方文档](https://huggingface.co/docs/transformers/main_classes/text_generation)、[Qwen3-8B 官方模型页](https://huggingface.co/Qwen/Qwen3-8B)，核对日期：2026-07-27。

## 理解检查

1. Top-k 和 Top-p 哪一个的候选数量固定？
2. 模型很确定时，Top-p 的候选集合为什么可能变小？
3. 这些参数为什么不应被理解成模型新增长期知识？

下一部分：[[00-停止与文本恢复概览|停止与文本恢复]]。
