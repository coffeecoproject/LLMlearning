---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Embedding向量表示概览]]"
previous: "[[初始表示与Hidden-State概览]]"
next: "[[Embedding参数与语义边界概览]]"
tags: [llm, embedding, parameters, tensor-shape, optional]
---

# Embedding 参数与深入

> [!summary]
> 这一层解释 Embedding Matrix 的形状和参数来源，并澄清“向量相似”与“模型真正理解语义”之间的边界。

## 阅读内容

1. [[Embedding参数与语义边界概览|Embedding 参数与语义边界]]；
2. [[形状词表规模与参数量|形状、词表规模与参数量]]；
3. [[从单个向量到序列张量|从单个向量到序列张量]]。

## 这一层回答什么

```text
vocab_size × hidden_size
→ 为什么决定 Embedding Matrix 的主要参数量

sequence_length × hidden_size
→ 为什么描述一个样本的向量序列

batch_size × sequence_length × hidden_size
→ 为什么描述一批整理后的模型输入
```

这些形状用于理解结构，不展开梯度推导、显存调度或服务端动态 Batch。

下一篇：[[Embedding参数与语义边界概览|Embedding 参数与语义边界]]。
