---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-MLA与注意力变体概览|MLA与注意力变体概览]]"
previous: "[[03-为什么MLA要解耦RoPE|为什么MLA要解耦RoPE]]"
next: "[[05-MLA与MHA-GQA-MQA有什么不同|MLA与MHA-GQA-MQA有什么不同]]"
tags: [llm, mla, attention-flow, forward-pass]
---

# MLA 完整流程

> [!summary]
> MLA 先从同一 Hidden State 形成 Query 路径、KV 内容 Latent 和单独的位置 Key，再完成内容匹配与位置匹配、Causal Mask、Softmax、Value 汇总和 Output Projection。

## 先看全景

以下是以 DeepSeek-V2/V3 MLA 为主的概念流程：

```text
输入 Hidden State
│
├── Query 向下投影 → Query Latent → 向上投影
│                                  ├→ Q_content
│                                  └→ Q_rope → RoPE
│
└── KV 向下投影
       ├→ KV Latent ───────────────┬→ K_content
       │                           └→ Value
       └→ K_rope ───────────────────→ RoPE

Q_content 与 K_content 匹配
+ Q_rope 与 K_rope 匹配
→ Score
→ Causal Mask
→ Softmax Weight
→ 汇总 Value
→ 多 Head Context
→ Output Projection
```

不要把图中的分支理解成完全互不联系的模型；它们由同一层参数组织，并在 Score 和 Context 阶段重新汇合。

## 第一步：接收本层 Hidden State

每个 Token 进入当前 Attention 子层时已有一条 Hidden State。MLA 没有重新 Tokenize，也不会把 Token ID 直接送进 Latent 压缩：

```text
Token ID
→ Embedding 与前面各层
→ 当前层 Hidden State
→ MLA
```

## 第二步：形成 Query 路径

DeepSeek-V3 配置了 Query 的低秩投影：

```text
Hidden State
→ 较窄 Query Latent
→ 展开为多个 Query Head
```

每个 Query Head 再拆为：

```text
Q_content：内容匹配分量
Q_rope：直接接受 RoPE 的位置分量
```

Query Latent 是当前 Query 的中间投影，不是历史 KV Cache。

## 第三步：形成 KV Latent 与位置 Key

同一个 Hidden State 还会进入 KV 向下投影，产生：

```text
KV Latent：联合支持内容 K 与 Value
K_rope：单独携带直接 RoPE 位置分量，并由各 Head 共享
```

概念上，KV Latent 可以继续向上展开为各 Head 的 `K_content` 与 `Value`。

## 第四步：计算两部分匹配

每个 Query Head 都会形成自己的总 Score：

```text
Q_content 与 K_content 的匹配
+ Q_rope 与 K_rope 的匹配
→ 该 Query Head 的总 Score
```

因此 MLA 仍保留多个 Query Head 的匹配路径；共享 Latent 不会把所有 Head 的 Weight 强制变成一样。

这里的共享 `K_rope` 仍是逐 Token 产生的：历史位置 0、1、2 各自有位置 Key，只是在同一位置上不为每个 Head 重复保留一份独立位置 Key。

## 第五步：执行 Causal Mask 与 Softmax

总 Score 后续仍走已经学过的步骤：

```text
Score
→ 排除未来位置
→ Softmax
→ 每个 Query Head 的 Attention Weight
```

MLA 没有取消 Decoder-only 模型的因果可见性规则。

## 第六步：汇总 Value 并输出

每个 Query Head 使用自己的 Weight 汇总相应 Value 信息，得到 Context：

```text
各 Query Head 的 Context
→ 合并
→ Output Projection
→ 一份 Attention 子层输出
```

它随后仍会进入 Residual、Normalization、FFN 等 Transformer Block 后续结构。

## 为什么代码里不一定看到“先完整展开 K/V”

上面的流程是便于理解的等价结构。因为向上投影是线性变换，优化实现可以调整矩阵乘法次序：

```text
概念路径：
KV Latent → 展开 K/V → Attention

优化路径：
把部分展开矩阵吸收到 Query 或 Output Projection
→ 直接围绕 KV Latent 完成等价计算
```

DeepSeek-V3 官方推理代码同时展示了较直观的 `naive` 路径和保存 `kv_cache`、`pe_cache` 的优化路径。这说明“模型数学结构”和“某个运行实现实际创建哪些中间张量”需要分开阅读。

## 阶段标注

> [!info] 训练阶段
> 完整前向流程用于计算训练预测；反向传播会更新 Query、KV 压缩/展开、位置路径与 Output Projection 等参数。训练并不是只学习一个独立压缩器后再拼到模型上。

> [!info] 普通运行阶段
> 参数固定，Prompt 的 Prefill 与后续 Decode 都使用 MLA 结构。运行实现可以用等价的矩阵吸收路径减少显式 K/V 展开和历史保存量。

## 来源

- [DeepSeek-V2 官方论文](https://arxiv.org/abs/2405.04434)，用于确认 MLA 的低秩联合压缩与解耦 RoPE 主线。
- [DeepSeek-V3 官方推理实现](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/inference/model.py)，用于确认 `naive` 与优化路径的实际连接。
- 核对日期：2026-07-27。

## 常见误解

- **“MLA 从 Token ID 直接生成 KV Latent。”** 它接收当前 Transformer 层的 Hidden State。
- **“有 KV Latent 后就不需要 Query。”** Query 仍决定当前位置如何匹配来源。
- **“MLA 不再做 Mask 和 Softmax。”** 这些核心步骤仍存在。
- **“概念图画了展开，所以所有 Runtime 都必须构造完整 K/V。”** 优化实现可以使用等价的矩阵吸收。
- **“Query Latent 会作为历史 KV 一起缓存。”** Query 路径与历史 K/V 保存职责不同。
- **“MLA 输出可以绕过 Transformer Block 的其余部分。”** 它仍是 Attention 子层。

## 理解检查

1. MLA 的三个关键中间来源分别是什么？
2. 内容分量与位置分量在哪一步重新汇合？
3. MLA 为什么仍然需要 Causal Mask 和 Softmax？
4. 为什么概念流程和优化代码中的显式中间张量可能不同？

下一篇：[[05-MLA与MHA-GQA-MQA有什么不同|MLA与MHA-GQA-MQA有什么不同]]。
