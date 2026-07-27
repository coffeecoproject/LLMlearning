---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Residual与Normalization概览|Residual与Normalization概览]]"
previous: "[[00-Residual与Normalization框架速览概览|Residual与Normalization框架速览概览]]"
next: "[[01-Residual-Connection是什么|Residual-Connection是什么]]"
tags: [llm, residual, transformer, mechanism]
---

# Residual 基础机制

> [!summary]
> 这一层把 `旧表示 + 子层变化` 放回 Attention、FFN 和完整 Transformer Block 中理解。

## 阅读顺序

1. [[01-Residual-Connection是什么|Residual Connection 是什么]]；
2. [[02-为什么不能只保留子层新结果|为什么不能只保留子层新结果]]；
3. [[03-Residual怎样连接Attention与FFN|Residual 怎样连接 Attention 与 FFN]]。

## 机制主线

```text
主干输入 x
├── 直接保留 x
└── 经过子层产生变化 F(x)

x + F(x)
→ 更新后的 Hidden State
```

这里的子层可以是 Attention，也可以是 FFN。Residual 不决定 Attention 看谁，也不负责 FFN 内部怎样加工。

## 相邻内容

Residual 解释“旧表示与子层变化怎样连接”，接下来的 [[00-Normalization基础机制概览|Normalization 基础机制]]解释向量尺度怎样整理，以及 LayerNorm、RMSNorm、Pre-Norm 与 Post-Norm 的区别。二者相邻但职责不同。

下一篇：[[01-Residual-Connection是什么|Residual Connection 是什么]]。
