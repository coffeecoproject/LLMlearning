---
type: topic-index
module: 1
status: complete
audience: non-specialist
parent: "[[01-LLM/01-基础结构与计算机制/基础结构与计算机制大纲]]"
previous: "[[Tokenizer文本离散化系统概览]]"
next: "[[Position位置机制概览]]"
tags: [llm, embedding, vector]
---

# Embedding 向量表示

> [!goal]
> 理解 Token ID 怎样通过 Embedding Matrix 变成模型可以计算的初始向量，并区分初始 Embedding 与上下文相关 Hidden State。

## 与 Tokenizer 的连接

```text
文本
→ Tokenizer
→ Token ID
→ Embedding Lookup
→ 每个位置的初始向量
```

Token ID 虽然是整数，但只是词表索引。ID `10001` 不比 `10000` “语义更大”，两者相差 1 也不表示对应 Token 语义接近。

> [!note]
> 上述 ID 仅为教学示意，不属于真实模型。

## 阶段标注

> [!info] 两阶段共同
> Embedding Lookup 是 LLM 前向计算的一部分：训练和运行都会根据 Token ID 取出对应向量。训练阶段允许误差信号更新 Embedding Matrix；普通运行阶段通常只读取已经训练好的矩阵，不会因为一次聊天自动改写它。

## 子结构与学习顺序

1. [[向量基础概览|向量基础]]：只学习理解 Embedding 所需的向量、维度、形状与相似性直觉。
2. [[Token-Embedding概览|Token Embedding]]：理解 Embedding Matrix 和 Lookup。
3. [[Embedding参数与语义边界概览|Embedding 参数与语义边界]]：认识 Embedding 是模型参数，但不等于完整词义。
4. [[初始表示与Hidden-State概览|初始表示与 Hidden State]]：区分初始向量与经过上下文处理的动态表示。

## 完整知识结构

```text
Embedding 向量表示
├── 01 向量基础
│   ├── 向量是什么
│   ├── 维度与 Hidden Size
│   ├── 向量距离与相似性
│   └── 从单个向量到序列张量
├── 02 Token Embedding
│   ├── Token ID 为什么不能直接计算语义
│   ├── Embedding Matrix
│   ├── Embedding Lookup
│   └── 形状、词表规模与参数量
├── 03 Embedding 参数与语义边界
└── 04 初始表示与 Hidden State
```

## 简单数字预览

假设：

```text
vocab_size = 5
hidden_size = 3
```

Embedding Matrix 的形状是：

```text
[5, 3]
```

意思是词表中 5 个 ID 各有一行 3 维向量。输入 ID `[2,4]` 会选出第 2 行和第 4 行，得到形状 `[2,3]` 的向量序列。本专题不要求展开矩阵推导。

## 本专题边界

- Position 不再混在 Embedding 中，独立进入 [[Position位置机制概览]]。
- Attention、FFN、Residual 与 Norm 进入 [[Transformer概览]]。
- Embedding 参数在训练中怎样被 Loss 和梯度更新，进入训练模块；这里仍保留“训练形成、运行读取”的必要边界。
- Batch 与 Padding 可能同时出现在训练和运行中；训练 Batch 怎样影响参数更新、运行 Batch 怎样合并用户请求，分别进入对应阶段模块。

## 完成标准

完成后应能：

1. 解释为什么 Token ID 已经是数字，却仍不能作为语义数值；
2. 用简单形状说明 Embedding Matrix 与 Lookup；
3. 区分 Token、Token ID、Token Embedding 和 Hidden State；
4. 说明 Embedding 是模型参数，但不等于固定、完整的词义。

下一专题：[[Position位置机制概览|Position 位置机制]]。
