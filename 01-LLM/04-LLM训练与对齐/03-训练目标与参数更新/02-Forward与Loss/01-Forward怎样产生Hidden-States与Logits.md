---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Forward与Loss概览|Forward 与 Loss 概览]]"
next: "[[02-Loss与Cross-Entropy怎样衡量预测偏差|Loss 与 Cross-Entropy 怎样衡量预测偏差]]"
tags: [llm, training, forward, hidden-state, logits]
---

# Forward 怎样产生 Hidden States 与 Logits

> [!summary]
> Forward 是使用当前模型参数完成一次从 Token ID 到 Logits 的计算；它展示模型此刻会怎样预测，但还没有评价预测好坏，也没有修改参数。

## Forward 是什么

Forward，中文常称前向计算，方向是：

```text
模型输入
→ 一层层向前计算
→ 模型输出
```

对于 Decoder-only LLM：

```text
input_ids
→ Token Embedding
→ Position Information
→ 多个 Transformer Block
→ Final Norm
→ LM Head
→ Logits
```

这条结构与普通运行基本相同。训练和运行的分叉发生在 Logits 之后。

## 为什么训练必须先 Forward

训练系统不能只看参数就知道当前模型对这个 Batch 的具体错误。

```text
当前参数
+ 当前输入
→ Forward
→ 当前预测
```

只有先得到当前预测，才能与 Labels 比较并计算 Loss。

## Hidden State 在这里是什么

每个 Token 位置经过各层 Transformer 后，会得到当前层对该位置的向量表示：

```text
[sequence_length, hidden_size]
```

加入 Batch 后：

```text
[batch_size, sequence_length, hidden_size]
```

这些 Hidden States 已经混合了允许读取的上下文信息，但它们还不是 Vocabulary 上的预测分数。

## LM Head 怎样得到 Logits

LM Head 把每个位置最后的 Hidden State 映射到 Vocabulary 大小：

```text
Hidden State：hidden_size 个数
→ LM Head
→ Logits：vocab_size 个数
```

如果：

```text
batch_size = 2
sequence_length = 4
vocab_size = 10000
```

Logits 的整体形状可写成：

```text
[2, 4, 10000]
```

含义是：2 条序列、每条 4 个位置、每个位置都有 10000 个候选 Token 的原始分数。

## 一个小例子

> [!example] 教学示意
> 假设 Vocabulary 只有三个候选 Token。

当前位置的 Logits：

```text
苹果： 2.0
香蕉： 1.0
汽车：-1.0
```

Logits 可以为负，也不需要加起来等于 1。Softmax 可以把它们转换成概率分布，但训练中的 Cross-Entropy 实现通常可以直接接收 Logits，以更稳定地完成组合计算。

## 训练会使用哪些位置的 Logits

训练通常为多个序列位置产生 Logits，但不一定全部计算 Loss：

```text
正文有效位置 → 可能参与 Loss
Padding 位置 → 通常忽略
SFT 的 User Prompt → 某些配方只读取、不计 Loss
没有下一目标的位置 → 按实现处理或忽略
```

因此：

```text
产生了 Logits
≠
这个位置一定影响参数更新
```

Labels 和 Loss Mask 决定哪些位置进入误差汇总。

## Forward 会修改参数吗

不会。

Forward 读取参数并产生中间结果：

```text
读取当前参数
→ 计算 Hidden States 和 Logits
```

训练框架还会保留 Backward 所需的计算关系或部分中间激活，但真正修改参数要等到 Optimizer Step。

## 训练 Forward 与普通运行 Forward 的差别

| 训练阶段 | 普通运行阶段 |
|---|---|
| 常对完整训练序列计算多个位置 | Prefill 后通常使用最后有效位置预测下一 Token |
| 通常需要保留 Backward 所需信息 | 不需要为参数训练保留反向图 |
| Logits 后进入 Loss | Logits 后进入 Token 选择 |
| 参数之后会由 Optimizer 更新 | 参数保持不变 |

## 常见误解

- **“Forward 就是生成答案。”** Forward 只产生当前预测分数；生成还需要 Token 选择和循环。
- **“Logits 就是概率。”** Logits 是 Softmax 前的原始分数。
- **“产生 Logits 后参数已经学习。”** 参数要到 Backward 和 Optimizer Step 后才改变。
- **“所有位置的 Logits 都一定计算 Loss。”** Padding 或被屏蔽位置通常不参与。

## 理解检查

1. 为什么训练必须先使用当前参数做 Forward？
2. Hidden State 与 Logits 分别处在哪个空间？
3. 为什么产生一个位置的 Logits，不代表该位置一定影响参数更新？
