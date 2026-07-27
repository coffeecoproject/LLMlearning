---
type: topic-index
module: 1
status: complete
audience: non-specialist
parent: "[[01-LLM/01-基础结构与计算机制/基础结构与计算机制大纲]]"
previous: "[[Embedding向量表示概览]]"
next: "[[Transformer概览]]"
tags: [llm, position, rope, alibi]
---

# Position 位置机制

> [!summary]
> Position 让模型计算能够利用 Token 的顺序与距离；不同位置方案可以作用在输入表示、Attention 的 Q/K 或 Attention Score。

## 按学习目标选择入口

### 只看框架

阅读：[[Position框架速览|Position 一页看懂]]。

只需分清：

```text
Embedding → 内容的初始表示
Position  → 顺序和距离
Mask      → 可见权限
```

### 理解基础机制

阅读：

1. [[为什么必须表示顺序]]；
2. [[三类位置机制对比]]。

先理解“为什么需要”和“三种方案作用在哪里”，不要求推导公式。

### 继续深入

按需阅读：

1. [[Absolute-Position-Embedding|Absolute Position Embedding]]；
2. [[RoPE的作用位置与直觉|RoPE 的作用位置与直觉]]；
3. [[ALiBi的作用位置与直觉|ALiBi 的作用位置与直觉]]。

具体旋转矩阵、长上下文缩放和外推评估不属于框架必读内容。

## 系统结构

```text
位置机制
├── 输入附近：Absolute Position Embedding
├── Q/K 关系：RoPE
└── Score 偏置：ALiBi
```

这是一张典型方案地图，不表示所有模型只能使用其中一种，也不表示三者构成连续流水线。

## 阶段与边界

> [!info] 两阶段共同
> 位置运算在 LLM 训练和运行时都会参与前向计算。可学习位置参数在训练中更新、运行时固定；固定规则仍会在两个阶段执行。

真实模型能否可靠利用更长上下文，还取决于训练范围、缩放方案、Attention 架构和评估结果。

框架路线下一站：[[Transformer框架速览|Transformer 一页看懂]]。
