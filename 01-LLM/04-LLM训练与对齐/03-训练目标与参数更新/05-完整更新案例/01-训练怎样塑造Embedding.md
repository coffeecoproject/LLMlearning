---
type: concept
module: 1
status: complete
audience: non-specialist
reading: optional
parent: "[[00-完整更新案例概览|完整更新案例概览]]"
next: "[[02-一次参数更新怎样完整发生|一次参数更新怎样完整发生]]"
tags: [llm, embedding, training, gradient, optimizer]
---

# 训练怎样塑造 Embedding

> [!summary] 一句话理解
> LLM 训练先用 Token ID 查出 Embedding，再根据整个模型的预测误差反向计算相关参数应怎样调整；Embedding 因而与 Attention、FFN 和输出层等参数一起逐步学习。

## 先分清两种训练

这里讨论的是 **LLM 参数训练**，不是 Tokenizer 训练：

```text
【Tokenizer 训练】
语料 → 学习词表和切分规则

【LLM 训练】
固定 Tokenizer → 文本变成 Token ID
→ 更新 Embedding、Attention、FFN 等模型参数
```

通常先确定 Tokenizer，再用稳定的 Token ID 映射训练 LLM。LLM 的 Backward 不会顺便重做 Token 切分规则。

## Embedding 位于完整训练链的哪里

```text
训练文本
→ 固定 Tokenizer
→ Token IDs
→ Embedding Lookup
→ Transformer Blocks
→ Logits
→ Loss
→ Backward
→ Embedding 等参数获得 Gradient
→ Optimizer 更新参数
```

Embedding 是模型参数体系的一部分，不是 Tokenizer 输出的固定语义答案。

## 第一步：Forward 使用当前 Embedding

假设训练片段是：

```text
我 喜欢 苹果
```

程序先根据每个 Token ID，从 Embedding Matrix 中取出相应参数行。取出的向量再进入 Transformer，最终参与下一 Token 预测。

如果当前参数还不理想，模型可能在“我 喜欢”之后给错误 Token 更高的 Logit。没有 Forward，就无法知道当前参数组合产生了什么预测。

## 第二步：Loss 把预测偏差变成数值目标

Loss 衡量预测与训练 Label 的偏差。直观上：

```text
正确目标概率低 → Loss 较高
正确目标概率高 → Loss 较低
```

Loss 不会直接说“苹果向量第 37 维应该加多少”。它只给整个可微计算过程提供一个需要降低的目标。

## 第三步：Backward 把影响追溯回来

Backward 从 Loss 出发，沿着刚才的计算路径反向追溯：

> 某个参数如果发生很小的变化，当前 Loss 会怎样变化？

这个局部方向与敏感程度用 Gradient（梯度）表示。Embedding 参与了 Forward，因此被 Lookup 的相关参数行可以收到梯度。

没有 Backward，最终预测误差就无法转换成输入端 Embedding 的更新依据。

## 第四步：Optimizer 真正修改参数

Optimizer 读取 Gradient，并结合 Learning Rate 等规则更新参数：

```text
当前 Embedding 参数
→ 根据梯度做一次小调整
→ 更新后的 Embedding 参数
```

Gradient 提供局部更新信息，Optimizer 执行更新；二者不是同一个概念。

## 为什么相似 Token 可能逐渐形成关系

假设“苹果”“香蕉”经常出现在相似上下文：

```text
我吃了一个苹果
我吃了一个香蕉
苹果是一种水果
香蕉是一种水果
```

它们对应的 Embedding 会反复参与相似预测任务，并与模型其他参数一起被调整。经过大量样本后，模型可能形成可复用的关系结构。

但这不是“每出现一次，就把两个向量直接拉近”：

- 优化目标是降低整个模型的预测 Loss；
- 相似关系是长期统计结果，不是人工写入的单条规则；
- 上下文语义还分布在 Transformer 的其他参数和动态 Hidden State 中；
- Token Embedding 接近，不等于任何语境下含义都相同。

## 是否只更新本批次出现的 Embedding 行

在标准 Lookup 的输入路径上，本 Batch 出现的 Token 行参与计算，可以从这条路径收到梯度；没有被查到的行通常不会从该输入路径收到直接梯度。

但许多模型会让输入 Embedding 与输出层共享权重，或者通过其他结构形成额外更新路径。因此不能脱离具体架构断言“一次训练只会影响出现过的几行”。

当前需要掌握的准确说法是：

> 被 Lookup 的 Embedding 行通过整个预测任务获得训练信号，且与其他模型参数联合学习。

## Batch 在这里做什么

一个 Batch 可以包含多条序列。模型汇总各有效位置的 Loss，并获得一次参数更新所需的整体梯度信号。

```text
多条训练序列
→ 批量 Forward
→ 汇总 Loss
→ Backward
→ Optimizer Step
```

如果使用 Gradient Accumulation，还可能积累多个 Micro-batch 后才执行一次 Optimizer Step。

## 一次更新后会发生什么

更新后的 Embedding 会在下一次 Forward 中被重新 Lookup。相同 Token ID 仍指向同一行，但该行的数值可能已经略有变化。

一次更新不会让它立刻“学会完整词义”。有用表示来自大量上下文、许多 Training Step 和模型各组件的联合训练。

## 常见误解

### 误解一：Tokenizer 训练会直接学出 LLM 的 Embedding

Tokenizer 确定词表、切分规则和 Token ID；Embedding 数值在 LLM 参数训练中学习。

### 误解二：Loss 会直接给出正确向量

Loss 只提供数值优化目标，Backward 和 Optimizer 才把它转化为参数变化。

### 误解三：相似上下文会把两个向量复制成一样

模型优化的是整体预测目标，不执行固定的向量复制规则。

### 误解四：Embedding 包含模型全部语义

Embedding 只是初始表示。具体上下文中的语义还经过 Attention、FFN 和多层 Block 形成在 Hidden State 中。

## 理解检查

1. 为什么 Tokenizer 固定后，Embedding 仍然可以继续学习？
2. Loss、Gradient 和 Optimizer 分别承担什么职责？
3. 为什么不能把“相似上下文”简化成“把两个 Embedding 行设成相同”？
4. 一次更新后，Token ID 会改变还是相应 Embedding 参数可能改变？

## 继续学习

- 返回：[[00-完整更新案例概览|完整更新案例概览]]
- 下一篇：[[02-一次参数更新怎样完整发生|一次参数更新怎样完整发生]]
