---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Attention扩展结构概览|Attention扩展结构概览]]"
previous: "[[00-MHA-GQA与MQA概览|MHA-GQA与MQA概览]]"
next: "[[00-Residual与Normalization概览|Residual与Normalization概览]]"
tags: [llm, mla, attention-variants, deepseek]
---

# MLA 与注意力变体

> [!summary]
> MLA（Multi-head Latent Attention）仍然使用多个 Query Head 完成匹配与信息汇总，但不把每个位置的完整多头 K/V 当作必须长期保留的表示；它先把 K/V 所需信息学习成较窄的潜在表示，再按 Attention 需要使用这些信息。

## 为什么把它放在标准 Attention 之后

前面已经建立了标准主线：

```text
Hidden State
→ Q、K、V
→ Score
→ Causal Mask
→ Softmax Weight
→ Context
→ 多 Head 合并
```

MLA 没有删除这条主线，而是重点改造其中这一段：

```text
标准 MHA：Hidden State → 完整多头 K、V
MLA：     Hidden State → 较窄的 KV 潜在表示 → Attention 所需的 K/V 信息
```

因此，理解 MLA 的前提不是新学一套完全不同的“注意力”，而是先知道标准 Q/K/V 路径，再看 K/V 表示和运行保存方式怎样被重新设计。

## 先看最简流程

```text
当前 Token 的 Hidden State
        │
        ├── Query 路径 ──────────────→ 多个 Query Head
        │
        └── KV 向下投影
                 ↓
          较窄的 KV Latent
                 ↓
          提供各 Head 所需的 K/V 信息
                 ↓
Query 与 Key 匹配 → Weight → 汇总 Value → Context
```

这里的 `Latent` 可以先理解为**模型学到的中间压缩表示**。它不是 Token ID，不是词向量，也不是把一整段文本压成一个向量；序列中的每个 Token、每个 MLA 层都有自己的相应潜在表示。

## 阅读顺序

1. [[01-MLA为什么需要压缩KV表示|MLA为什么需要压缩KV表示]]：先理解它要解决的结构问题。
2. [[02-低秩联合压缩怎样表示K与V|低秩联合压缩怎样表示K与V]]：理解“低秩”“联合”和“潜在表示”分别指什么。
3. [[03-为什么MLA要解耦RoPE|为什么MLA要解耦RoPE]]：理解位置信息为什么要使用一条单独路径。
4. [[04-MLA完整流程|MLA完整流程]]：把 Query、KV Latent、RoPE、Score、Context 串成一条因果链。
5. [[05-MLA与MHA-GQA-MQA有什么不同|MLA与MHA-GQA-MQA有什么不同]]：分清“减少 KV Head”和“压缩 KV 表示”。
6. [[06-怎样阅读DeepSeek-V3的MLA配置|怎样阅读DeepSeek-V3的MLA配置]]：读取真实模型的维度和官方实现。
7. [[07-其他注意力变体地图|其他注意力变体地图]]：按“究竟改了 Attention 的哪一部分”认识其他常见名称。

## 三个必须先守住的边界

### MLA 不等于 MQA

MQA 主要通过让所有 Query Head 共享一组 K/V Head 来减少 K/V；MLA 的核心是学习一个较窄的 KV 潜在表示，并从中提供多头 Attention 所需的信息。两者都可能减少运行资源，但结构方法不同。

### MLA 不等于普通文件压缩

它不是先生成完整 K/V，再调用通用压缩程序打包。压缩和展开投影本身就是模型参数的一部分，会在训练中共同学习。

### MLA 不等于某个推理框架优化

MLA 是模型结构，训练阶段与运行阶段都会经过这套表示路径。不同运行实现可以选择显式恢复完整 K/V，也可以通过矩阵吸收直接围绕潜在表示计算；这是结构与实现之间的区别。

## 阶段标注

> [!info] 两阶段共同
> MLA 的低秩投影、位置分支和多头计算都属于模型前向结构。训练阶段学习这些投影参数；普通运行阶段读取固定参数并执行相同的函数关系。

> [!note] 运行阶段动机
> MLA 的重要动机是减少自回归生成期间需要为历史位置保留的信息量。本专题解释“为什么结构上可以这样做”；KV Cache 的总量计算、Paged Attention、内存带宽、Prefill 与 Decode 性能仍留到普通运行模块。

## 开放权重模型观察

DeepSeek-V2 官方论文系统提出 MLA；DeepSeek-V3 官方技术报告说明 V3 延续采用 MLA，并沿用 V2 的核心设计。DeepSeek-V3 的公开配置与官方推理代码进一步展示了 `q_lora_rank`、`kv_lora_rank`、非位置 Q/K 维度和 RoPE Q/K 维度等真实字段。

来源：

- [DeepSeek-V2 官方论文](https://arxiv.org/abs/2405.04434)
- [DeepSeek-V3 官方技术报告](https://arxiv.org/abs/2412.19437)
- [DeepSeek-V3 官方推理实现](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/inference/model.py)
- 核对日期：2026-07-27。

## 完成标准

学完后应能：

1. 指出 MLA 改造了标准 Attention 的哪段路径；
2. 解释 KV Latent 为什么不是一个 Token，也不是整段文本的单一摘要；
3. 用小维度例子说明低秩向下投影和向上投影；
4. 解释为什么 MLA 要把 RoPE 位置分量单独处理；
5. 区分 MLA、MHA、GQA、MQA 与 FlashAttention；
6. 从 DeepSeek-V3 配置识别 MLA 的关键字段；
7. 分开描述模型结构、训练参数学习和运行缓存实现。

下一篇：[[01-MLA为什么需要压缩KV表示|MLA为什么需要压缩KV表示]]。
