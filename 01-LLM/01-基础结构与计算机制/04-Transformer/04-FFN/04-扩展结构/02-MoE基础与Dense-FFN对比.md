---
type: reference
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN扩展结构概览|FFN扩展结构概览]]"
previous: "[[01-SwiGLU与GeGLU|SwiGLU与GeGLU]]"
next: "[[03-Router-Expert与Top-k|Router-Expert与Top-k]]"
tags: [llm, ffn, moe, dense, expert]
---

# MoE 基础与 Dense FFN 对比

> [!summary]
> Dense FFN 让每个 Token 使用同一套完整 FFN；MoE 准备多套 Expert FFN，并由 Router 为每个 Token 选择少数 Expert。

## 它替换 Block 中哪一块

Dense 基线：

```text
Attention → Residual → Dense FFN → Residual
```

MoE 路线：

```text
Attention → Residual
→ Router + 多个 Expert FFN
→ 合并 Expert 输出
→ Residual
```

MoE 主要扩展 FFN/MLP 子层，不是 MHA、GQA、MLA 那一类 Attention 变体。

## 直接对比

```text
Dense FFN
Token A ─┐
Token B ─┼→ 同一套完整 FFN
Token C ─┘

MoE
每个 Token → Router → 选择少数 Expert FFN
```

## Expert 到底是什么

一个 Expert 通常是一套 FFN/MLP 参数路径，内部仍可执行升维、SwiGLU 和降维。它通常没有自己完整的 Tokenizer、Embedding、几十层 Transformer 和 Output Layer。

所以：

```text
多个 Expert
≠ 多个完整 LLM 开会投票
```

## 为什么叫稀疏激活

假设某层存有 8 个 Expert，每个 Token 只选择 2 个：

```text
模型存储：8 个 Expert
本 Token 主要执行：2 个 Expert
```

稀疏的是当前 Expert 计算路径，不表示未选参数不存在，也不表示 Hidden State 必须是稀疏向量。

## 为什么设计 MoE

MoE 希望做到：

```text
增加模型可以容纳的参数容量
同时避免每个 Token 执行所有 Expert
```

代价是引入 Router 学习、负载平衡、权重存储、跨设备通信和复杂部署。

## MoE 后面的主线没有消失

选中 Expert 的输出合并后仍需回到 `hidden_size`，再加入 Residual。后续 Block、Final Norm 和 Output Layer 都照常存在。

## 常见误解

- Expert 不是完整 Agent，也不是完整 LLM。
- MoE 不代表所有 Expert 都同时计算。
- 未选择的 Expert 仍占模型存储。
- Expert 职责未必能命名成“数学”“中文”等人类学科。
- MoE 不是外部工具路由。

下一篇：[[03-Router-Expert与Top-k|Router、Expert 与 Top-k]]。
