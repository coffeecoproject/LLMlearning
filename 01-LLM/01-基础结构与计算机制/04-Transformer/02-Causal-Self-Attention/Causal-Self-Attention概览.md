---
type: topic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Transformer概览]]"
previous: "[[Transformer整体结构概览]]"
next: "[[Residual与Normalization概览]]"
tags: [llm, attention, self-attention, causal-attention]
---

# Causal Self-Attention

> [!summary]
> Causal Self-Attention 让每个位置从自己和过去的可见位置汇集信息，同时禁止读取未来位置。

## 按学习目标选择入口

### 只看框架

阅读：[[Attention框架速览概览|Attention 一页看懂]]。

只需理解：

```text
当前位置提出匹配需求
→ 在可见位置中分配信息份量
→ 汇总相应内容
```

### 理解基础机制

进入：[[Attention基础机制概览|Attention 基础机制]]。

这里完整连接 Q、K、V、Score、Mask、Weight、Context 和 Multi-Head Attention。

### 继续深入

进入：[[Attention扩展结构概览|Attention 扩展结构]]。

这里比较 MHA、GQA、MQA、MLA，以及模型结构与 KV Cache 运行实现之间的边界。

## 基础计算链

```text
Hidden States
→ Q、K、V
→ Score
→ Scaling / 位置影响 / Causal Mask
→ Softmax 与 Weight
→ Value 加权汇总
→ 多 Head 合并与 Output Projection
→ Attention 子层输出
```

## 阶段边界

> [!info] 两阶段共同
> 训练与运行都会执行 Causal Self-Attention。训练时 Mask 防止完整训练序列泄漏未来答案；运行时保持相同的左到右依赖关系。

本专题不展开 Loss、梯度、完整逐 Token 生成、KV Cache 内存管理或服务请求调度。

## 当前内容

- [x] [[Attention框架速览概览|框架速览]]
- [x] [[Attention基础机制概览|基础机制]]
- [x] [[Attention扩展结构概览|扩展结构]]

框架路线下一站：[[Residual与Normalization框架速览概览|Residual 与 Normalization 一页看懂]]。
