---
type: subsection-index
module: 1
status: planned
audience: non-specialist
parent: "[[Causal-Self-Attention概览]]"
previous: "[[MHA-GQA与MQA概览]]"
next: "[[Residual与Normalization概览]]"
tags: [llm, mla, attention-variants, deepseek]
---

# MLA 与注意力变体

> [!summary]
> MLA 等注意力变体不会改变“根据当前位置寻找并汇总上下文”这一根本职责，而是重新设计 Q、K、V 的表示、共享或缓存方式，以改变能力与效率之间的取舍。

## 计划结构

1. 先用 Multi-Head Attention（MHA）、Grouped Query Attention（GQA）和 Multi-Query Attention（MQA）作为已经掌握的基线；
2. MLA 改变了标准 Head 与 KV 表示中的什么；
3. 为什么不能把 MLA 简化成“只是另一种 GQA”；
4. QK-Norm、局部或滑动窗口注意力等只做位置地图，不在基础主线逐一深挖；
5. 把模型结构变化与运行时 KV Cache 优化分开说明。

> [!note] 扩展阅读边界
> 本节用于读懂真实开放模型，不作为理解标准 Attention 的前置条件。压缩维度、旋转位置分量和具体矩阵实现放入可选技术部分。

## 开放模型观察：DeepSeek-V3

DeepSeek-V3 官方仓库说明该模型采用 Multi-head Latent Attention（MLA）与 DeepSeekMoE。这里先记录“它属于真实模型采用的架构分支”，具体机制等完成标准 MHA、GQA 与 MQA 后再展开。

来源：[DeepSeek-V3 官方仓库](https://github.com/deepseek-ai/DeepSeek-V3)，核对日期：2026-07-27。
