---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer基础机制概览|Output-Layer基础机制概览]]"
previous: "[[04-Softmax怎样把分数变成概率|Softmax怎样把分数变成概率]]"
next: "[[00-Output-Layer参数与深入概览|Output-Layer参数与深入概览]]"
tags: [llm, logits, training, inference, next-token]
---

# 训练与运行怎样使用 Logits

> [!summary]
> 训练和运行都会经过同一条“Hidden State → LM Head → Logits”前向路线；差别主要发生在 Logits 产生之后：训练用它衡量预测误差，运行用它选出下一个 Token。

## 两阶段共同部分

```text
输入 Token ID
→ Embedding
→ Transformer
→ Final Norm
→ LM Head
→ Logits
```

普通运行时参数固定；训练时参数之后会根据 Loss 更新。但本次前向计算中的组件关系基本一致。

## LLM 训练阶段：多个位置用于学习

使用一个教学序列：

```text
输入位置： 我 ｜ 喜欢 ｜ 猫
学习目标：喜欢 ｜ 猫   ｜ <EOS>
```

含义是：

```text
看到“我”之后，预测“喜欢”
看到“我 喜欢”之后，预测“猫”
看到“我 喜欢 猫”之后，预测“<EOS>”
```

模型可以并行算出各位置的 Logits，再把有效位置与向后错开一位的目标 Token 对照，形成 Loss。Padding 或不应参与训练的部分会通过 Mask 等方式排除。

> [!warning] 示例边界
> 真实 Token 切分和特殊 Token 由具体 Tokenizer 与训练格式决定。这个例子只说明“当前位置学习预测下一个位置”。

## LLM 运行阶段：当前最后位置用于续写

用户输入：

```text
我喜欢
```

模型完成 Prompt 的前向计算后，生成系统通常读取当前最后一个有效位置的词表 Logits：

```text
“喜欢”所在位置的 Logits
→ 选择下一个 Token，例如“猫”
```

随后把新 Token 作为下一轮输入：它仍会经过 Embedding 和全部 Transformer Blocks，并在 Attention 中复用历史 KV Cache；Output Layer 只在这一轮末尾把新的 Final Hidden State 转换为下一轮 Logits。完整的逐 Token 循环、KV Cache、采样和停止条件属于“单请求推理与生成”，本节只保留接口。

## 为什么两者看起来不同

训练数据已经包含后续 Token，因此可以同时为许多位置计算监督信号；真实运行时未来 Token 尚不存在，只能先选出一个，再继续下一步。

这不是说训练时模型可以违反 Causal Mask 偷看未来。训练会并行计算多个位置，但每个位置仍只能利用它被允许看到的过去与当前位置。

## Batch 中有 Padding 怎么办

当不同序列补齐成相同长度时，系统会根据 Attention Mask、有效长度或标签 Mask 判断哪些位置有效：

- 训练：Padding 位置通常不计入 Loss；
- 运行：应读取每条序列当前的最后一个有效位置，而不是误把 Padding 当作真实内容。

具体 Padding 方向和 Runtime 实现属于运行工程，本节只需理解“最后一个有效位置”而不是“数组中无条件最后一格”。

## 最后对比

| 问题 | 训练阶段 | 运行阶段 |
|---|---|---|
| 是否产生 Logits | 是 | 是 |
| 常用哪些位置 | 多个有效位置 | 当前最后一个有效位置用于下一步 |
| Logits 后面做什么 | 计算 Loss | 选择下一个 Token |
| 参数是否更新 | 会在反向传播和优化后更新 | 普通运行中不更新 |

## 理解检查

1. 训练为什么能同时利用多个位置，却没有偷看未来？
2. 运行时为什么重点读取最后一个有效位置？
3. Padding 位置为什么不能被当作真实训练目标？

继续阅读：[[00-Output-Layer参数与深入概览|Output Layer 参数与深入]]。
