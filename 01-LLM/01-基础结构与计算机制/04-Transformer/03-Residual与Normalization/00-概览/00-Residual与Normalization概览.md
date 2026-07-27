---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Transformer概览|Transformer概览]]"
previous: "[[00-Causal-Self-Attention概览|Causal-Self-Attention概览]]"
next: "[[00-FFN概览|FFN概览]]"
tags: [llm, residual, layernorm, rmsnorm]
---

# Residual 与 Normalization

> [!summary]
> Residual 管理“旧表示与子层变化怎样连接”，Normalization 管理“向量数值尺度怎样整理”；二者共同组织 Transformer Block 的数据流。

## 按学习目标选择入口

### 只看框架

阅读：[[00-Residual与Normalization框架速览概览|Residual 与 Normalization 一页看懂]]。

只需掌握：

```text
Residual → 保留主干并加入变化
Normalization → 调整数值尺度
```

### 理解基础机制

1. [[00-Residual基础机制概览|Residual 基础机制]]：理解 `x + F(x)` 及两次 Residual；
2. [[00-Normalization基础机制概览|Normalization 基础机制]]：理解 LayerNorm、RMSNorm、Pre/Post-Norm 和 Final Norm。

### 简单数学与真实配置

- [[07-Normalization小数字示例|Normalization 小数字示例]]使用一个三维向量做演示；
- [[03-RMSNorm是什么|RMSNorm]]中使用 Qwen3-8B 官方配置观察 `rms_norm_eps` 与实际结构。

这些不是框架路线的前置条件。

## Block 中的位置

```text
Block 输入
→ Norm → Attention → Residual
→ Norm → FFN       → Residual
→ Block 输出
```

这是常见 Pre-Norm 串行结构的简化图。Post-Norm、并行子层和其他 Norm 变体会改变具体排列。

## 概念分类

```text
Residual
→ 连接与相加关系

LayerNorm / RMSNorm
→ Normalization 的计算规则

Pre-Norm / Post-Norm
→ Norm 在子层与 Residual 附近的放置位置

Final Norm
→ Block 堆叠结束、LM Head 之前的归一化
```

## 阶段边界

> [!info] 两阶段共同
> Residual 与 Normalization 前向计算在训练和运行时都会发生。训练阶段还会更新 Norm 的可学习参数并涉及梯度稳定性；普通运行阶段只使用固定参数。

## 当前进度

- [x] [[00-Residual与Normalization框架速览概览|框架速览]]
- [x] [[00-Residual基础机制概览|Residual 基础机制]]
- [x] [[00-Normalization基础机制概览|Normalization 基础机制]]

下一节：[[00-FFN概览|FFN / MLP]]。
