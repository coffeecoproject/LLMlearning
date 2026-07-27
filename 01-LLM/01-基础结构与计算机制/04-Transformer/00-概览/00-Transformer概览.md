---
type: topic-index
module: 1
status: complete
audience: non-specialist
parent: "[[01-LLM/01-基础结构与计算机制/00-概览/00-基础结构与计算机制大纲|基础结构与计算机制大纲]]"
previous: "[[00-Position位置机制概览|Position位置机制概览]]"
next: "[[00-Output-Layer输出层概览|Output-Layer输出层概览]]"
tags: [llm, transformer, attention, ffn, residual, normalization]
---

# Transformer

> [!summary]
> Transformer 通过多层 Block 反复更新各 Token 位置的 Hidden State；Attention、FFN、Residual 和 Normalization 在每个 Block 内分工协作。

## 按学习目标选择入口

### 只看整体框架

阅读：[[01-Transformer框架速览|Transformer 一页看懂]]。

然后按同样的框架路线依次阅读：

1. [[00-Attention框架速览概览|Attention 一页看懂]]；
2. [[00-Residual与Normalization框架速览概览|Residual 与 Normalization 一页看懂]]；
3. [[00-FFN框架速览概览|FFN 一页看懂]]；
4. [[00-Block堆叠框架速览概览|Block 堆叠一页看懂]]。

### 理解基础机制

1. [[00-Transformer整体结构概览|Transformer 整体结构]]；
2. [[00-Attention基础机制概览|Attention 基础机制]]；
3. [[00-Residual基础机制概览|Residual 基础机制]]；
4. [[00-Normalization基础机制概览|Normalization 基础机制]]；
5. [[00-FFN基础机制概览|FFN 基础机制]]；
6. [[00-Block堆叠与Hidden-State流动概览|Block 堆叠与 Hidden State 流动]]；
7. [[00-完整Transformer-Block串联复习概览|完整 Transformer Block 串联复习]]。

### 继续深入

- [[00-Attention扩展结构概览|Attention 扩展结构]]：MHA、GQA、MQA、MLA 的进一步比较；
- [[00-FFN参数与深入概览|FFN 参数与深入]]与[[00-FFN扩展结构概览|FFN 扩展结构]]；
- [[04-LayerNorm与RMSNorm对比|LayerNorm 与 RMSNorm 对比]]、[[05-Pre-Norm与Post-Norm|Pre-Norm 与 Post-Norm]]与[[07-Normalization小数字示例|小数字示例]]；
- [[00-Block参数与边界概览|Block 参数与边界]]：层数、参数、阶段边界与开放模型观察。

## 真实系统结构

```text
Transformer 主体
├── Block 1
│   ├── Attention + Residual / Norm
│   └── FFN       + Residual / Norm
├── Block 2
│   ├── Attention + Residual / Norm
│   └── FFN       + Residual / Norm
└── ……更多 Block
```

学习顺序与系统结构不同：我们会分专题学习组件，但真实前向计算是在每个 Block 中反复组合它们。

## 阶段边界

> [!info] 两阶段共同
> Transformer 前向计算在 LLM 训练和运行时都会发生。训练阶段还会根据误差更新参数；普通运行阶段只使用固定参数计算当前 Hidden States。

KV Cache、请求 Batch、采样和部署调度属于运行系统，不混入 Transformer 基础结构主线。

## 当前进度

- [x] [[01-Transformer框架速览|整体框架]]
- [x] [[00-Causal-Self-Attention概览|Causal Self-Attention]]
- [x] [[00-Residual与Normalization概览|Residual 与 Normalization]]
- [x] [[00-FFN概览|FFN / MLP]]
- [x] [[00-Block堆叠与Hidden-State流动概览|Block 堆叠与 Hidden State 流动]]
- [x] [[00-完整Transformer-Block串联复习概览|完整 Block 串联复习]]

Transformer 基础结构已经闭环。下一专题：[[00-Output-Layer输出层概览|Output Layer 输出层]]。
