---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[Embedding向量表示概览]]"
tags: [llm, token-embedding, embedding-matrix, lookup]
---

# Token Embedding

> [!summary]
> Token Embedding 使用 Token ID 作为行号，从 Embedding Matrix 中取出对应的初始向量；ID 是地址，不是向量的数值含义。

## 子结构

1. [[Token-ID为什么不能直接计算语义|Token ID 为什么不能直接计算语义]] ✓
2. [[Embedding-Matrix|Embedding Matrix]] ✓
3. [[Embedding-Lookup|Embedding Lookup]] ✓
4. [[形状词表规模与参数量|形状、词表规模与参数量]] ✓

## 核心链路

```text
Token ID
→ 作为 Embedding Matrix 的行索引
→ 取出 hidden_size 个数
→ 得到该 Token 的初始向量
```

## 边界

本节讲“怎样查出初始向量”，不提前解释 Attention 怎样改变它，也不展开 LM Head 是否与 Embedding 权重共享。
