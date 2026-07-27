---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN概览|FFN概览]]"
previous: "[[00-FFN框架速览概览|FFN框架速览概览]]"
next: "[[01-FFN的直白理解|FFN的直白理解]]"
tags: [llm, ffn, mechanism]
---

# FFN 基础机制

> [!goal]
> 在已经知道 FFN 的整体位置后，进一步理解它怎样加工 Hidden State，以及为什么这种加工不能由 Attention 完全代替。

## 这一层包含什么

1. [[01-FFN的直白理解|FFN 的直白理解]]
2. [[02-FFN在Transformer-Block中的位置与关联|FFN 在 Transformer Block 中的位置与关联]]
3. [[03-FFN核心机制与关键名词|FFN 核心机制与关键名词]]
4. [[04-FFN小数字完整计算|FFN 小数字完整计算]]

## 推荐阅读方式

前三篇解释直觉、位置和机制，属于这一层的主线。

第四篇用极小数字演示“展开 → 非线性 → 压回 → Residual”，用于把抽象机制落到具体数值；如果暂时不想看计算，可以先跳过，不影响继续理解整体结构。

```text
框架位置
→ 与 Attention、Residual 的关系
→ 展开、非线性、压回
→ 可选的小数字示例
```

## 这一层暂时不承担什么

- 不要求计算真实模型参数量；
- 不展开 Batch 和完整 Tensor 形状；
- 不讨论 MoE 与 Expert 路由；
- 不要求记忆真实模型配置。

这些内容已经分别放在 [[00-FFN参数与深入概览|参数与深入]]、[[00-FFN扩展结构概览|扩展结构]] 和 [[00-FFN真实模型观察概览|真实模型观察]] 中。

## 学完应能解释

- FFN 为什么位于 Attention 之后；
- FFN 为什么逐位置处理却仍能利用上下文；
- 展开、非线性、压回分别解决什么问题；
- FFN 输出为什么能够通过 Residual 加回主干。

下一篇：[[01-FFN的直白理解|FFN 的直白理解]]。
