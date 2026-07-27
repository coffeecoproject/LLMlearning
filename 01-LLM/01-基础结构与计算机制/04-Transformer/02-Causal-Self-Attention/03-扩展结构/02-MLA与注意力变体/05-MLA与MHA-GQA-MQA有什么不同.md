---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-MLA与注意力变体概览|MLA与注意力变体概览]]"
previous: "[[04-MLA完整流程|MLA完整流程]]"
next: "[[06-怎样阅读DeepSeek-V3的MLA配置|怎样阅读DeepSeek-V3的MLA配置]]"
tags: [llm, mla, mha, gqa, mqa, comparison]
---

# MLA 与 MHA、GQA、MQA 有什么不同

> [!summary]
> MHA、GQA、MQA 主要改变 Query Head 与 KV Head 的数量对应关系；MLA 则通过低秩潜在空间重新组织 K/V 内容表示，并把 RoPE 位置分量单独处理，因此不能只靠“有几个 KV Head”描述完整机制。

## 四者各自在改什么

```text
标准 MHA
→ 每个 Query Head 配一组独立 K/V Head

GQA
→ 一组 Query Head 共享一个 KV Head

MQA
→ 所有 Query Head 共享一个 KV Head

MLA
→ K/V 内容共同来自较窄 KV Latent
→ 位置 Key 使用解耦 RoPE 路径
→ 多个 Query Head 仍分别完成匹配
```

前三者最容易沿着“KV Head 有多少个”理解；MLA 必须再加入“KV 信息通过什么中间表示产生”这一维度。

## 一张结构对比表

| 问题 | 标准 MHA | GQA | MQA | MLA（以 DeepSeek 为主） |
|---|---|---|---|---|
| 多个 Query Head | 保留 | 保留 | 保留 | 保留 |
| K/V 资源取舍方式 | 每个 Q Head 独立 K/V | 组内共享 K/V Head | 全部共享一组 K/V | K/V 内容联合压缩到 Latent |
| 是否靠减少 KV Head 为核心 | 否 | 是 | 是 | 不是核心描述 |
| 是否有学习到的 KV 低秩瓶颈 | 标准结构没有 | 标准定义没有 | 标准定义没有 | 有 |
| 是否专门解耦 RoPE 分量 | 标准定义没有 | 标准定义没有 | 标准定义没有 | DeepSeek MLA 有 |
| 是否仍产生多套 Query Weight | 是 | 是 | 是 | 是 |
| 是否仍有 Causal Mask、Softmax、Context | 是 | 是 | 是 | 是 |

## 为什么 MLA 不是“更极端的 MQA”

MQA 的关系可以画成：

```text
Q0、Q1、Q2、Q3 → 同一 KV Head
```

MLA 更接近：

```text
每个位置的 Hidden State
→ 共享的紧凑 KV Latent
→ 为多头匹配提供 K/V 内容信息
```

MQA 明确让多 Query 共享一组完整 K/V Head；MLA 让多头 K/V 内容受到共同潜在表示的约束，但仍可通过不同向上投影形成不同 Head 的内容表示。

## 一个容易踩坑的配置判断

在常见 GQA/MQA 模型中，可以先比较：

```text
num_attention_heads
num_key_value_heads
```

但 DeepSeek-V3 的官方配置同时给出：

```text
num_attention_heads = 128
num_key_value_heads = 128
kv_lora_rank = 512
qk_nope_head_dim = 128
qk_rope_head_dim = 64
```

如果只看前两个相等，就可能把它误判成普通标准 MHA。自定义模型类、`kv_lora_rank` 和解耦位置字段共同表明这里使用的是 MLA 路径。

所以正确原则是：

```text
简单字段规则用于初步判断
自定义架构必须结合模型类、完整配置与实现
```

## 它们是否可以在同一模型里组合

这些名称不是天然互斥的所有维度。例如一个模型还可以同时具有：

```text
某种 Q/K/V Head 组织
+ 某种位置编码
+ 局部或全局可见范围
+ 某种高效 Attention 内核
```

但不能仅凭概念上“可以组合”，就断言某个具体模型真的采用了某组合；需要查官方配置和实现。

## 资源与质量边界

MHA、GQA、MQA、MLA 都体现表示能力与运行资源之间的结构设计。不能从名称直接推出：

- 固定的回答质量排名；
- 固定的速度倍数；
- 固定的最大上下文长度；
- 对所有硬件都相同的收益。

DeepSeek-V2 论文报告的是其模型与实验条件下的 KV Cache、吞吐和质量结果，不是“任何 MLA 都自动得到同一数值”。

## 来源

- [DeepSeek-V2 官方论文](https://arxiv.org/abs/2405.04434)，包含 MHA、GQA、MQA、MLA 的结构图与消融比较。
- [DeepSeek-V3 官方技术报告](https://arxiv.org/abs/2412.19437)，确认 DeepSeek-V3 延续 MLA。
- 核对日期：2026-07-27。

## 常见误解

- **“MLA 就是 KV Head 更少的 GQA。”** 它的核心是低秩联合压缩与位置分支。
- **“`num_attention_heads = num_key_value_heads` 就能证明标准 MHA。”** 对自定义 MLA 配置不够。
- **“共享 Latent 表示所有 Head 的 K/V 完全相同。”** 不同向上投影仍能形成不同多头内容表示。
- **“MLA 一定替代所有其他注意力设计。”** 不同模型会选择不同维度的组合。
- **“架构名能直接决定模型质量。”** 质量还取决于规模、数据、训练和其他组件。

## 理解检查

1. GQA/MQA 与 MLA 分别主要改变哪一层关系？
2. MLA 为什么不能简单画成“所有 Query 共用一个完整 KV Head”？
3. 为什么 DeepSeek-V3 的两个 Head 数量相等仍不足以证明它是标准 MHA？
4. Attention 架构名称为什么不能直接推出固定速度倍数？

下一篇：[[06-怎样阅读DeepSeek-V3的MLA配置|怎样阅读DeepSeek-V3的MLA配置]]。
