---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Backward与参数更新概览|Backward 与参数更新概览]]"
previous: "[[01-Backward与Gradient分别是什么|Backward 与 Gradient 分别是什么]]"
next: "[[../04-训练进度与状态/00-训练进度与状态概览|训练进度与状态概览]]"
tags: [llm, training, optimizer, learning-rate, adamw]
---

# Optimizer 与 Learning Rate 怎样更新参数

> [!summary] 一句话理解
> Gradient 告诉训练程序参数应朝哪个方向调整；Optimizer 决定具体怎样调整，Learning Rate 控制这一步大致走多远。

## 所属阶段

**训练阶段。** 普通运行只读取参数，不执行 Optimizer Step。

## 它接在什么位置

```text
Forward → Loss → Backward → Gradient → Optimizer Step → 新参数
```

- **Backward**：计算各参数对 Loss 的影响方向和程度，结果是 Gradient。
- **Optimizer Step**：根据 Gradient 真正修改参数。

所以 Gradient 不是新参数，它只是更新依据。

## 一个最简单的数字例子

下面是帮助理解方向的简化公式，不是 AdamW 的完整算法：

```text
新参数 = 旧参数 - Learning Rate × Gradient
```

假设：

```text
旧参数 = 1.00
Gradient = +0.40
Learning Rate = 0.10
```

那么：

```text
新参数 = 1.00 - 0.10 × 0.40 = 0.96
```

这个例子只需要看懂两点：

1. 通常沿 Gradient 的反方向移动，尝试降低 Loss；
2. Learning Rate 决定这一步的尺度。

## Learning Rate 为什么重要

Learning Rate（学习率）过大，参数可能一步跨过较好的区域，使 Loss 剧烈波动甚至训练失败；过小则更新缓慢，训练成本很高。

因此常见训练会调整学习率：

- **Warmup**：训练刚开始时，从较小步幅逐渐升高；
- **Learning Rate Schedule**：训练过程中按计划改变或逐渐降低步幅。

学习率不是模型学到的知识，而是训练程序控制更新节奏的设置。

## Optimizer 是什么

Optimizer（优化器）是参数更新算法。它读取 Gradient，并根据自己的规则计算本轮每个参数应改变多少。

### SGD

SGD（Stochastic Gradient Descent，随机梯度下降）的直觉最接近前面的简化公式：主要根据当前 Gradient 更新。

### AdamW

AdamW 是大模型训练中常见的优化器。它还会参考前几轮 Gradient 的统计信息，为不同参数调整更新尺度，并配合 Weight Decay 控制参数。

初学阶段不必推导公式，只需保留这条主线：

> AdamW 没有理解文本，它只是把数值 Gradient 转换成更稳定的参数改变量。

## 更新后模型为什么会变化

一次 Optimizer Step 往往会修改大量参数。下一次 Forward 读取新参数，因此：

```text
Embedding、Attention、FFN 等计算略有变化
→ Hidden States 略有变化
→ Logits 与概率分布略有变化
```

单次改变通常很小。经过大量不同数据和许多 Step，模型才逐渐形成语言、知识与任务能力。

这不是向数据库写入一条独立知识，而是许多参数共同改变了预测倾向。

## 选读：训练稳定性与恢复

- **Weight Decay**：对选定参数施加轻微收缩约束，避免参数规模无约束增大。
- **Gradient Clipping**：在更新前限制过大的 Gradient，降低某一步突然失控的风险。
- **Optimizer State**：AdamW 会保存历史 Gradient 的统计状态；若要尽量原样恢复训练，通常不仅要保存模型权重，还要保存这些状态。
- **清零 Gradient**：完成一次更新后，训练程序通常清除本轮 Gradient，再开始下一轮；有意跨多个小批累积时则另行控制。

这些内容解释训练系统怎样稳定运行，不改变 LLM 的下一 Token 训练目标。

## 常见误解

### Optimizer 会判断答案是否合理

不会。它只处理 Gradient、参数和更新规则。

### Learning Rate 就是模型掌握知识的速度

不完全是。它只控制更新尺度；数据、模型规模、Batch、优化器和算力等也会影响训练。

### 每次更新只修改一个“答错的参数”

不是。一次 Loss 通常会为大量参与计算的参数产生 Gradient。

## 理解检查

1. Backward 与 Optimizer Step 的职责有什么区别？
2. Learning Rate 过大或过小分别可能怎样？
3. 为什么一次更新不能等同于写入一条知识？

## 继续学习

- 上一篇：[[01-Backward与Gradient分别是什么|Backward 与 Gradient 分别是什么]]
- 下一部分：[[../04-训练进度与状态/00-训练进度与状态概览|训练进度与状态概览]]
