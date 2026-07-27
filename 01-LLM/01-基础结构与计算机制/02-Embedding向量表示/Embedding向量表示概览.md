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

> [!summary]
> Embedding 把 Token ID 转换成初始向量，使离散符号能够进入 Transformer 的连续数值计算。

## 按学习目标选择入口

### 只看框架

阅读：[[Embedding框架速览概览|Embedding 一页看懂]]。

只需掌握：

```text
Token ID → 查表 → 初始向量
```

### 理解基础机制

进入：[[Embedding基础机制概览|Embedding 基础机制]]。

这里解释向量的最低必要知识、Embedding Matrix、Lookup，以及初始表示与 Hidden State 的关系。

### 继续深入

进入：[[Embedding参数与深入概览|Embedding 参数与深入]]。

这里讨论形状、参数量、序列张量和语义空间边界。

## 系统结构

```text
Embedding
├── 输入：Token ID
├── 核心资源：Embedding Matrix
├── 操作：按 ID 查找对应行
├── 输出：每个 Token 位置的初始向量
└── 下游：Position 与 Transformer Hidden State 主干
```

## 阶段边界

> [!info] 两阶段共同
> LLM 训练和运行都会执行 Embedding Lookup。训练阶段可以更新矩阵参数，普通运行阶段只读取固定参数。

Batch、Padding 和部署调度不是 Embedding 的核心机制；它们只在解释输入形状时作为边界出现。

## 当前内容

- [x] [[Embedding框架速览概览|框架速览]]
- [x] [[Embedding基础机制概览|基础机制]]
- [x] [[Embedding参数与深入概览|参数与深入]]

框架路线下一站：[[Position框架速览|Position 一页看懂]]。
