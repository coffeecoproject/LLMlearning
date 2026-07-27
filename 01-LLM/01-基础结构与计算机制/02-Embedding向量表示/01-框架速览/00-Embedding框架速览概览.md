---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Embedding向量表示概览|Embedding向量表示概览]]"
previous: "[[00-Embedding向量表示概览|Embedding向量表示概览]]"
next: "[[00-Embedding基础机制概览|Embedding基础机制概览]]"
tags: [llm, embedding, framework, beginner]
---

# Embedding 一页看懂

> [!summary]
> Embedding 使用 Token ID 从一张训练形成的表中取出向量，把离散编号转换成 Transformer 可以继续计算的初始表示。

## 它位于哪里

```text
文字
→ Tokenizer
→ Token ID
→ Embedding Lookup
→ 每个 Token 位置的初始向量
→ Transformer
```

## Token ID 已经是数字，为什么还要转换

Token ID 只是词表中的行号：

```text
ID 10001
```

不表示它比 ID 10000 “多一个单位的语义”，也不表示两个 ID 对应的 Token 更相似。

Embedding 为每个 ID 提供一整组可以参与模型计算的数值：

```text
Token ID → 查表 → 向量
```

## 一个教学例子

```text
ID 2 → [0.2, -0.1, 0.7]
ID 4 → [0.6,  0.3, 0.1]
```

这些数字只是人为示意。真实模型通常使用数千维向量，具体数值由训练形成。

## 它输出什么

一句话有多个 Token，Embedding 会为每个位置取出一个向量：

```text
[ID1, ID2, ID3]
→ [向量1, 向量2, 向量3]
```

这些是进入 Transformer 前的初始表示。经过 Transformer 各层更新后，它们会成为上下文相关的 Hidden States。

## 训练与运行

> [!info] 两阶段共同
> LLM 训练和运行都会执行 Embedding Lookup。

```text
LLM 训练阶段
→ Embedding Matrix 可以随误差信号更新

LLM 运行阶段
→ 读取已经训练好的矩阵，不因一次聊天自动改写
```

## 它没有做什么

- Embedding 不负责切分文本；
- 查表不是按照相似度搜索最近向量；
- 初始 Token Embedding 不是完整、固定的词义；
- Embedding 本身没有完成上下文理解；
- Token ID 不是先转换成某种通用文本编码再进入 Embedding。

## 框架层检查

1. Embedding 的输入和输出分别是什么？
2. 为什么不能直接拿 Token ID 的数值大小表示语义？
3. 初始 Token Embedding 与后续 Hidden State 有什么区别？

能回答这三题，就可以先进入 [[01-Position框架速览|Position 一页看懂]]。需要继续理解向量、查表和 Hidden State 时，再读 [[00-Embedding基础机制概览|Embedding 基础机制]]。
