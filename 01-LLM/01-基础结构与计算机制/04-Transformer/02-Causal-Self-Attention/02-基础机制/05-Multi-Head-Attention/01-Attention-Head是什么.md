---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Multi-Head-Attention概览|Multi-Head-Attention概览]]"
previous: "[[00-Multi-Head-Attention概览|Multi-Head-Attention概览]]"
next: "[[02-为什么需要多个Head|为什么需要多个Head]]"
tags: [llm, attention-head, qkv, context]
---

# Attention Head 是什么

> [!summary]
> 一个 Attention Head 是一条从输入表示产生自己的一组 Q、K、V，再计算匹配权重并汇总 Value 的 Attention 计算路径。

## 从已经学过的单头流程理解

前面学习的这条链，本身就可以看成一个 Head：

```text
输入 Hidden States
→ Q、K、V
→ 对 Q、K 施加位置变换（若采用 RoPE 等方案）
→ Score 与 Scaling
→ 加入 Score 位置偏置（若采用 ALiBi 等方案）
→ Causal Mask
→ Attention Weight
→ Value 加权求和
→ Context
```

Multi-Head Attention 并没有发明另一种 Attention 公式，而是让多条这样的路径并行存在。

## Head 的输入和输出

对于一条长度为 `S` 的序列，省略 Batch 维度后，一个 Head 可以抽象为：

```text
输入：整条序列各位置的 Hidden States
Q：   [S, head_dim]
K：   [S, head_dim]
V：   [S, head_dim]
输出：[S, head_dim]
```

输出仍然按位置排列：序列中每个接收位置都会得到一个属于该 Head 的 Context Vector。Head 不是为整句话只生成一个共同摘要。

## 为什么不同 Head 会不同

多个 Head 虽然读取同一组输入 Hidden States，但它们使用不同的可学习投影参数，或等价地使用大投影结果中的不同参数分区：

```text
Head 1：H → Q₁、K₁、V₁
Head 2：H → Q₂、K₂、V₂
```

因此同一个位置在两个 Head 中可能得到不同的：

- Query；
- Key；
- Value；
- Score；
- Attention Weight；
- Context。

差异不是系统人为写入“Head 1 看语法、Head 2 看人物”，而是训练过程中参数逐渐形成的。

## Head 不是什么

一个 Head 不是：

- 一个 Token；
- 一个完整 Transformer Layer；
- 一个独立小模型；
- 一段自然语言规则；
- 一个必然具有固定人类语义的“视角”；
- 一次完整的推理步骤。

“Head 关注某种关系”可以作为事后观察，但不能把某个 Head 永久命名为唯一功能模块。同一个 Head 的行为会随层、输入和上下文改变。

## 一个简化类比及其边界

可以暂时把多个 Head 类比为几位同时阅读同一句话、但使用不同检查重点的读者。

类比有助于理解“并行观察”，但到这里就停止匹配：真实 Head 没有意识，也不会用语言写出检查意见。它执行的是可学习线性投影、数值匹配、Softmax 和向量加权求和。

## 与 Transformer Layer 的关系

```text
多个 Attention Head
→ 合并为 Multi-Head Attention 输出
→ 进入 Residual / Normalization 等结构
→ 再进入 FFN
```

所以 Head 是 Attention 子层内部的一条路径，而 Attention 子层又只是 Transformer Block 的一部分。

## 常见误解

- **“一个 Head 只看一个 Token。”** 一个 Head 会为每个接收位置比较所有允许读取的来源位置。
- **“一个 Head 对应一句话中的一个含义。”** Head 输出是每个位置的向量序列，不是一个自然语言标签。
- **“Head 越多就一定越聪明。”** Head 数量只是结构选择，能力还取决于维度、数据、训练和整个模型。
- **“不同 Head 使用不同输入文本。”** 它们从同一组输入表示出发，但投影参数和内部表示不同。

## 理解检查

1. 为什么一个 Head 可以被称为一条完整的 Attention 路径？
2. 两个 Head 读取同一 Hidden State，为什么仍可能得到不同 Weight？
3. 一个 Head 与一个 Transformer Layer 有什么区别？
4. 为什么不能把某个 Head 永久认定为“语法 Head”？

下一篇：[[02-为什么需要多个Head|为什么需要多个Head]]。
