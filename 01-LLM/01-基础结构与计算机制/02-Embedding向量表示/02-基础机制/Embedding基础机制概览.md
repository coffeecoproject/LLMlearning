---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Embedding向量表示概览]]"
previous: "[[Embedding框架速览概览]]"
next: "[[向量基础概览]]"
tags: [llm, embedding, vector, hidden-state]
---

# Embedding 基础机制

> [!summary]
> 这一层解释向量是什么、Embedding Lookup 怎样工作，以及初始 Token Embedding 怎样进入 Hidden State 主干。

## 阅读顺序

1. [[向量基础概览|向量基础]]：只补足理解向量、维度和序列表示所需知识；
2. [[Token-Embedding概览|Token Embedding]]：理解 Embedding Matrix 与 Lookup；
3. [[初始表示与Hidden-State概览|初始表示与 Hidden State]]：理解初始表示怎样被 Transformer 逐层更新。

## 机制主线

```text
Token ID
→ 在 Embedding Matrix 中选取对应行
→ 得到每个位置的初始向量
→ 组成向量序列
→ 进入 Transformer
→ 逐层形成上下文相关 Hidden States
```

基础路线只需理解“查表”和“表示逐层变化”。矩阵参数量、Batch 形状和语义空间边界可进入参数深入层。

下一篇：[[向量基础概览|向量基础]]。
