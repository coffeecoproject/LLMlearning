---
type: topic-index
module: 1
status: complete
audience: non-specialist
parent: "[[01-LLM/01-基础结构与计算机制/00-概览/00-基础结构与计算机制大纲|基础结构与计算机制大纲]]"
previous: "[[00-Transformer概览|Transformer概览]]"
next: "[[00-Output-Layer框架速览概览|Output-Layer框架速览概览]]"
tags: [llm, output-layer, lm-head, logits, softmax]
---

# Output Layer 输出层

> [!summary]
> Output Layer 把 Transformer 最终形成的上下文表示转换成“词表中每个 Token 的候选分数”；这些分数叫 Logits，它们随后在训练阶段用于计算 Loss，在运行阶段用于选择下一个 Token。

## 它在整条路线中的位置

```text
文字
→ Tokenizer
→ Token ID
→ Embedding 与 Position
→ 多层 Transformer Block
→ Final Hidden States
→ Final Norm
→ LM Head
→ Logits
```

到 `Logits` 为止，仍是模型的一次前向计算。之后才根据阶段分叉：

```text
训练阶段
→ 使用多个有效位置的 Logits 计算 Next Token Loss
→ 反向传播更新参数

运行阶段
→ 使用当前最后一个有效位置的 Logits
→ 调整分数并形成概率
→ 选择下一个 Token
```

## 分层结构

### Level 0：只建立框架

- [[00-Output-Layer框架速览概览|Output Layer 一页看懂]]

读完应能回答：Output Layer 的输入、主要动作、输出和边界分别是什么？

### Level 1：理解基础机制

- [[00-Output-Layer基础机制概览|Output Layer 基础机制]]
- [[01-Final-Hidden-State怎样进入输出层|Final Hidden State 怎样进入输出层]]
- [[02-LM-Head是什么|LM Head 是什么]]
- [[03-Logits是什么|Logits 是什么]]
- [[04-Softmax怎样把分数变成概率|Softmax 怎样把分数变成概率]]
- [[05-训练与运行怎样使用Logits|训练与运行怎样使用 Logits]]

### Level 2：参数与简单计算（可选）

- [[00-Output-Layer参数与深入概览|Output Layer 参数与深入]]
- [[01-输出形状与LM-Head参数量|输出形状与 LM Head 参数量]]
- [[02-Weight-Tying是什么|Weight Tying 是什么]]
- [[03-Logits到概率的小数字示例|Logits 到概率的小数字示例]]
- [[04-真实模型Output-Layer观察|真实模型 Output Layer 观察]]

### Level 3：边界与复习

- [[00-Output-Layer边界与复习概览|Output Layer 边界与复习]]
- [[01-Output-Layer不等于生成策略|Output Layer 不等于生成策略]]
- [[02-从文本输入到Logits完整串联|从文本输入到 Logits 完整串联]]
- [[03-Output-Layer理解检查|Output Layer 理解检查]]

## 这一专题最重要的因果链

```text
Hidden State 只有 hidden_size 个内部特征
→ 还不能直接对应整个词表
→ LM Head 为词表中的每个 Token 计算一个分数
→ 得到 vocab_size 个 Logits
→ 训练目标或运行策略再使用这些分数
```

## 必须分清的四个对象

| 对象 | 它是什么 | 它不是什么 |
|---|---|---|
| Final Hidden State | 某个位置的最终上下文向量 | Token ID 或概率 |
| LM Head | 从 `hidden_size` 映射到 `vocab_size` 的可学习投影 | 采样算法 |
| Logits | 每个候选 Token 的原始分数 | 已归一化概率 |
| Probability | Logits 经 Softmax 等处理后形成的相对概率 | 模型对事实正确性的置信度 |

## 专题边界

这里讲清：

- Final Norm 与 LM Head 怎样连接；
- 为什么输出宽度等于 `vocab_size`；
- Logits 与概率有什么区别；
- 训练与运行怎样使用同一类输出。

这里不深入：

- Cross Entropy、反向传播和优化器，进入“LLM 训练与对齐”；
- Temperature、Top-k、Top-p、采样、停止条件和 KV Cache，进入“单请求推理与生成”；
- API 服务怎样调度多个用户请求，属于 Runtime 与服务系统。

建议从 [[00-Output-Layer框架速览概览|Output Layer 一页看懂]] 开始。
