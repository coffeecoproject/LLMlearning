---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-训练目标与参数更新概览|训练目标与参数更新概览]]"
previous: "[[00-Forward与Loss概览|Forward 与 Loss 概览]]"
next: "[[00-训练进度与状态概览|训练进度与状态概览]]"
tags: [llm, backward, gradient, optimizer, learning-rate]
---

# Backward 与参数更新概览

> [!summary]
> Backward 沿 Forward 的计算关系反向计算 Gradient，Optimizer 再决定怎样使用这些 Gradient 修改参数。

## 主线

```text
Loss
→ Backward
→ 每个相关参数的 Gradient
→ 可选的 Gradient 处理
→ Optimizer Step
→ 更新后的参数
```

## 两个核心问题

1. [[01-Backward与Gradient分别是什么|Backward 与 Gradient 分别是什么]]
2. [[02-Optimizer与Learning-Rate怎样更新参数|Optimizer 与 Learning Rate 怎样更新参数]]

## 边界

- Backward 主要负责计算 Gradient，不负责更新参数。
- Optimizer 更新参数，但不会自行判断训练数据是否真实可靠。
- Learning Rate 太大可能让训练不稳定，太小可能让学习进展缓慢。
- 混合精度、分布式通信与训练稳定性工程放到“规模化训练系统”。
