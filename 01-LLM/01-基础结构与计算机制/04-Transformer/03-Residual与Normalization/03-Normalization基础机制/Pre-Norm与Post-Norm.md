---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[Normalization基础机制概览]]"
previous: "[[LayerNorm与RMSNorm对比]]"
next: "[[Block内Norm与Final-Norm]]"
tags: [llm, pre-norm, post-norm, residual, transformer-block]
---

# Pre-Norm 与 Post-Norm

> [!summary]
> Pre-Norm 在 Attention 或 FFN 子层之前执行 Norm；Post-Norm 先执行子层和 Residual 相加，再对相加结果做 Norm。

## Pre-Norm

对一个子层 `F`，常见 Pre-Norm 关系是：

```text
x
├───────────────┐
└→ Norm → F ────┘
        ↓
      相加
        ↓
y = x + F(Norm(x))
```

Norm 处理的是进入子层分支的数据；Residual 主干 `x` 直接保留。

常见串行 Block：

```text
x
→ Norm → Attention → Residual
→ Norm → FFN       → Residual
→ y
```

## Post-Norm

常见 Post-Norm 关系是：

```text
x
├───────────┐
└→ F ───────┘
      ↓
    相加
      ↓
    Norm
      ↓
y = Norm(x + F(x))
```

Norm 处理的是子层输出与主干相加后的结果。

## 为什么位置差异重要

两种结构改变了：

- 子层接收到的是原始主干还是已经归一化的输入；
- Residual 主干是否直接跨过 Norm；
- 深层网络的梯度与数值传播方式；
- 是否通常需要堆叠结束后的 Final Norm。

训练稳定性的严格分析进入训练模块；基础结构阶段先能画出两条数据流。

## 它们与 LayerNorm / RMSNorm 是不同问题

```text
LayerNorm / RMSNorm
→ 用什么规则整理数值

Pre-Norm / Post-Norm
→ 在子层与 Residual 的什么位置整理
```

不要把 Pre-Norm 理解成一种新的归一化公式。

## 真实模型还有其他变体

有些架构使用：

- 并行 Attention 与 FFN；
- 额外的 Sandwich Norm；
- 针对 Q、K 的局部 Norm；
- 不同 Residual 缩放方式。

因此这里的两张图是常见基础类型，不是所有模型的唯一实现。

## 阶段边界

> [!info] 两阶段共同
> 模型架构确定 Norm 的放置位置后，训练和运行都按同一数据流执行。普通运行不会临时把 Pre-Norm 改成 Post-Norm。

## 常见误解

- Pre/Post 描述位置，不描述训练前后阶段。
- Pre-Norm 不是“预训练时才用的 Norm”。
- Post-Norm 不是“模型输出后只执行一次”。
- 具体模型结构必须看对应版本实现。

## 理解检查

1. `x + F(Norm(x))` 属于哪种结构？
2. `Norm(x + F(x))` 属于哪种结构？
3. 为什么 Pre-Norm 与 RMSNorm 不是同一分类维度？

下一篇：[[Block内Norm与Final-Norm|Block 内 Norm 与 Final Norm]]。
