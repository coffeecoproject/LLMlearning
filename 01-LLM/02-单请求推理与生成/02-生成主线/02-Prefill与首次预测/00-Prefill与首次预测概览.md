---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-单请求生成主线概览|单请求生成主线概览]]"
previous: "[[02-Tokenizer与模型输入张量|Tokenizer与模型输入张量]]"
next: "[[01-Prefill是什么|Prefill是什么]]"
tags: [llm, inference, prefill, logits]
---

# Prefill 与首次预测概览

> [!summary]
> Prefill 是模型在开始回答前，对当前完整输入序列做的一次前向处理：形成各位置表示、建立可复用的 Attention 状态，并得到第一个新 Token 的候选分数。

## 它位于哪里

```text
输入 Token 序列
→ Prefill
├── 首次下一 Token Logits
└── 各层 KV Cache
→ 选择第一个回答 Token
```

## 阅读顺序

1. [[01-Prefill是什么|Prefill 是什么]]；
2. [[02-为什么读取最后有效位置的Logits|为什么读取最后有效位置的 Logits]]。

## 先记住一个关键区别

- Prefill 的输入通常包含多个已有 Token；这些位置可以在硬件上进行大量并行计算，但仍受 Causal Mask 限制。
- 后续 Decode 每一步通常只新增一个 Token，下一步又依赖刚刚选出的 Token，因此生成方向具有串行依赖。

下一篇：[[01-Prefill是什么|Prefill 是什么]]。
