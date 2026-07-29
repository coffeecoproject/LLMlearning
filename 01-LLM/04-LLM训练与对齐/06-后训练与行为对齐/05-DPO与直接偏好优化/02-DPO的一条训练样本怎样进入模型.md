---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化概览]]"
previous: "[[01-为什么会出现DPO|为什么会出现 DPO]]"
next: "[[03-DPO怎样比较Chosen与Rejected的生成倾向|DPO 怎样比较 Chosen 与 Rejected 的生成倾向]]"
tags: [llm, dpo, preference-pair, tokenization, loss-mask]
---

# DPO 的一条训练样本怎样进入模型

> [!summary] 一句话理解
> DPO 把同一个 Prompt 分别与 Chosen、Rejected 组成两条序列，使用同一 Tokenizer 转成 Token IDs，再只统计回答部分的生成倾向用于偏好比较。

## 原始数据长什么样

```text
Prompt：请用一句话解释 Embedding。

Chosen：Embedding 把离散 Token 映射成模型内部使用的连续向量。

Rejected：Embedding 就是把每个单词换成一个随机数字。
```

三个字段承担不同角色：

- Prompt：两条回答共同面对的条件；
- Chosen：相对更受偏好的候选；
- Rejected：相对较差的候选。

## 第一步：应用相同 Chat Template

对话模型可能把输入整理成：

```text
System：……
User：请用一句话解释 Embedding。
Assistant：候选回答
```

Chosen 与 Rejected 必须使用相同角色结构和 Prompt。否则模型比较的就不只是回答差异。

Chat Template 属于输入格式；DPO 本身并不规定所有模型使用同一种特殊 Token。

## 第二步：组成两条训练序列

```text
序列 A = Prompt + Chosen
序列 B = Prompt + Rejected
```

它们不是拼成一条让模型继续阅读的自然对话，而是作为两个候选分别计算。

## 第三步：Tokenizer 转成 Token IDs

```text
序列 A → Tokenizer → Token IDs A
序列 B → Tokenizer → Token IDs B
```

这里使用模型配套的已训练 Tokenizer。DPO 通常不会在这一步重新构建 Vocabulary 或重新训练 BPE。

不同回答长度可以通过 Padding 和 Attention Mask 组织成 Batch；Padding 不属于回答内容。

## 第四步：区分 Prompt 与回答 Token

模型计算回答概率时，Prompt 提供条件，但偏好 Loss 通常重点统计 Assistant 回答位置。

```text
Prompt Token：作为上下文输入
Response Token：计算 Chosen / Rejected 的序列倾向
Padding Token：排除
```

可以把它看成与 SFT 的 Loss Mask 类似：输入部分帮助模型理解任务，但不会被误当成需要比较的候选回答。

具体库对 BOS、EOS、对话边界和截断位置的处理可能不同，必须遵循模型的 Chat Template 和训练配置。

## 第五步：同一批数据送给两个模型角色

原始 DPO 通常需要：

```text
Current Policy
→ 分别计算 Chosen 与 Rejected

Frozen Reference Model
→ 分别计算 Chosen 与 Rejected
```

因此一条偏好对在概念上会得到四个序列倾向：

1. Policy 对 Chosen 的倾向；
2. Policy 对 Rejected 的倾向；
3. Reference 对 Chosen 的倾向；
4. Reference 对 Rejected 的倾向。

下一篇会解释为什么需要四个，而不是只看 Policy 的两个结果。

## Truncation 为什么需要特别小心

如果长度超过训练上限，系统可能截断序列。错误截断可能导致：

- Prompt 关键约束丢失；
- Chosen 与 Rejected 的关键差异被截掉；
- EOS 或回答末尾缺失；
- 两条候选比较失去原本含义。

因此偏好数据管线需要明确最大长度、截断方向和过滤规则。这属于数据构造质量，不是 DPO Loss 能自动修复的问题。

## 一个完整的可观察过程

```text
偏好记录
→ Chat Template
→ Prompt+Chosen / Prompt+Rejected
→ Tokenizer
→ Token IDs 与 Masks
→ Policy Forward
→ Reference Forward
→ 四个回答倾向
→ DPO Loss
```

## 常见误解

### Chosen 和 Rejected 是两个特殊 Token

不是。它们是数据角色标签，对应两条候选回答。

### Prompt 完全不进入模型

错误。Prompt 是判断回答是否合适的条件，只是偏好比较通常统计回答 Token 的生成倾向。

### DPO 会同时更新 Policy 和 Reference

原始 DPO 通常只更新 Policy，Reference 保持冻结。

## 理解检查

1. 为什么 Chosen 与 Rejected 必须共享相同 Prompt？
2. Prompt Token 为什么要进入模型，却通常不作为候选回答部分统计？
3. 一条偏好对为什么会产生四个模型—回答组合？
4. Truncation 可能怎样破坏偏好样本？

## 继续学习

- 上一篇：[[01-为什么会出现DPO|为什么会出现 DPO]]
- 下一篇：[[03-DPO怎样比较Chosen与Rejected的生成倾向|DPO 怎样比较 Chosen 与 Rejected 的生成倾向]]
