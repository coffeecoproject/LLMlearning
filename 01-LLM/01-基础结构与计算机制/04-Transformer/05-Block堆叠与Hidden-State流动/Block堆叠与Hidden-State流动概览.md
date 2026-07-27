---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[Transformer概览]]"
previous: "[[FFN概览]]"
next: "[[Output-Layer输出层概览]]"
tags: [llm, transformer-block, hidden-state, layers]
---

# Block 堆叠与 Hidden State 流动

> [!summary]
> 每个 Transformer Block 接收上一层 Hidden States，完成一轮 Attention、FFN、Residual 和 Normalization 组合更新，再把同宽度的新 Hidden States 交给下一层。

## 按学习目标选择入口

### 只看框架

阅读：[[Block堆叠框架速览概览|Block 堆叠一页看懂]]。

只需掌握：

```text
初始表示
→ Block 1 更新
→ Block 2 再更新
→ ……
→ Final Hidden States
```

### 理解基础机制

进入：[[Block堆叠基础机制概览|Block 堆叠基础机制]]。

这里解释 Block 与 Layer、逐层 Hidden State、堆叠原因、主干形状以及最后一层输出。

### 继续深入

进入：[[Block参数与边界概览|Block 参数与边界]]。

这里讨论 `num_hidden_layers`、每层参数、开放模型层数，以及 Hidden State 在训练和运行阶段的不同生命周期。

## 系统结构

```text
Transformer 主体
├── Block 1：H⁰ → H¹
├── Block 2：H¹ → H²
├── Block 3：H² → H³
├── ……
└── Block N：Hᴺ⁻¹ → Hᴺ

Hᴺ
→ Final Norm
→ Output Layer
```

`H` 在这里表示整条序列的 Hidden States，不是 `hidden_size` 的缩写；上标只表示经过了第几层，不是乘方。

## 阶段边界

> [!info] 两阶段共同
> LLM 训练和运行都会逐层计算 Hidden States。训练阶段还要保留或重新计算部分中间状态用于反向传播；普通运行阶段参数固定，中间状态是当前请求的临时计算结果。

Block 堆叠属于模型结构。KV Cache、连续批处理和跨设备流水线并行属于运行或部署实现，不在基础机制中展开。

## 当前内容

- [x] [[Block堆叠框架速览概览|框架速览]]
- [x] [[Block堆叠基础机制概览|基础机制]]
- [x] [[Block参数与边界概览|参数与边界]]

框架路线下一站：[[Output-Layer输出层概览|Output Layer 输出层]]。
