---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[Transformer概览]]"
previous: "[[Block堆叠与Hidden-State流动概览]]"
next: "[[一次Block完整数据流]]"
tags: [llm, transformer-block, review, data-flow]
---

# 完整 Transformer Block 串联复习

> [!summary]
> 这一节不再引入新的主要组件，而是把 Position、Attention、Normalization、Residual、FFN、Block 堆叠和 Final Norm 接成一条完整数据流。

## 复习顺序

1. [[一次Block完整数据流|一次 Block 完整数据流]]：沿一个常见 Pre-Norm Block 逐步走完；
2. [[完整Block形状追踪|完整 Block 形状追踪]]：用 `[1,3,4]` 主干和 `intermediate_size=8` 检查形状；
3. [[Transformer基础结构最终检查|Transformer 基础结构最终检查]]：检查是否能独立解释整个 Transformer。

## 完整路线预览

```text
Token ID
→ Token Embedding
→ 初始 Hidden States H⁰

位置影响按模型方案进入：
├── Absolute Position：输入表示附近
├── RoPE：Attention 的 Q/K
└── ALiBi：Attention Score

→ Block 1
   → Norm
   → Causal Self-Attention
   → Residual
   → Norm
   → FFN
   → Residual
→ H¹

→ Block 2 …… Block N
→ Final Hidden States Hᴺ
→ Final Norm
→ Output Layer / LM Head
→ Logits
```

## 复习边界

这条路线讲模型的一次前向结构，不展开：

- Loss、反向传播和优化器；
- KV Cache、连续 Batch 和服务调度；
- Temperature、Top-k、Top-p 与采样；
- Agent 的工具、记忆和执行循环。

这些系统会使用或围绕 Transformer，但不属于一个 Block 的内部结构。

下一篇：[[一次Block完整数据流|一次 Block 完整数据流]]。
