---
type: framework-note
module: 1
status: complete
audience: non-specialist
parent: "[[00-Transformer概览|Transformer概览]]"
previous: "[[00-Transformer概览|Transformer概览]]"
next: "[[00-Transformer整体结构概览|Transformer整体结构概览]]"
tags: [llm, transformer, block, framework, beginner]
---

# Transformer 一页看懂

> [!summary]
> Transformer 是 LLM 的主要计算主体：它把每个 Token 位置的初始表示反复送入多个 Block，让表示逐层吸收上下文并得到更新。

## 它位于哪里

```text
Token ID
→ Embedding 与位置机制
→ Transformer Block × 多层
→ Final Hidden States
→ Output Layer
```

## 为什么 Embedding 后还需要 Transformer

Embedding 只为每个 Token ID 提供初始向量。同一个“苹果”出现在“吃苹果”和“苹果股票”中，初始 Token Embedding 可以相同，但当前上下文含义不同。

Transformer 通过多层计算不断更新这些位置的表示，使其逐渐带上当前上下文影响。

## 一个 Block 最少包含什么

```text
Attention
→ 让不同 Token 位置交流信息

FFN
→ 分别加工每个位置已经取得的信息

Residual
→ 保留主干并加入子层产生的变化

Normalization
→ 调整数值尺度，帮助深层数据流稳定
```

所以 Transformer 不等于 Attention，Attention 只是 Block 内的一个核心子层。

## 多层在做什么

```text
初始表示
→ Block 1 更新
→ Block 2 再更新
→ ……
→ Final Hidden States
```

常见情况下 Token 位置数量和主干宽度不会因为经过一个 Block 就改变，但向量内部数值已经更新。

## Decoder-only 是什么意思

现代生成式 LLM 常采用带 Causal 约束的 Decoder-only Transformer：每个位置只能利用自己和允许读取的过去位置，然后模型从左到右继续生成。

这里的 Decoder 与 Tokenizer 的 `decode()` 不是同一概念。Decoder-only 仍然需要 Tokenizer 编码用户输入。

## 框架层检查

1. Transformer 位于 Embedding 与 Output Layer 之间做什么？
2. Attention、FFN、Residual、Normalization 各承担什么职责？
3. 为什么经过 Block 后形状可以不变，表示内容却已经变化？
4. Decoder-only 为什么不表示“没有编码过程”？

能回答这四题，就已经掌握 Transformer 的整体框架。下一步可以进入 [[00-Attention框架速览概览|Attention 一页看懂]]。
