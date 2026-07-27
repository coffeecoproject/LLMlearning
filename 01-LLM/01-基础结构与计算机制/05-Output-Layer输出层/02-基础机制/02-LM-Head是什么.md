---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer基础机制概览|Output-Layer基础机制概览]]"
previous: "[[01-Final-Hidden-State怎样进入输出层|Final-Hidden-State怎样进入输出层]]"
next: "[[03-Logits是什么|Logits是什么]]"
tags: [llm, lm-head, linear-projection, vocabulary]
---

# LM Head 是什么

> [!summary]
> LM Head 是连接“模型内部 Hidden State 空间”和“Tokenizer 词表空间”的可学习投影：它为词表中的每个 Token 计算一个原始分数。

## 先直白理解

一个 Hidden State 可能有 `hidden_size` 个数，但模型需要对 `vocab_size` 个候选 Token 打分：

```text
一个 hidden_size 维向量
→ LM Head
→ vocab_size 个候选分数
```

LM 是 Language Model，Head 可以理解为接在模型主体末端、面向某种任务输出的部件。这里的任务是语言建模，因此通常称为 `LM Head`。

## 为什么需要这次映射

Hidden State 的维度表达的是训练形成的内部特征，不是一份词表清单。即使 `hidden_size=4096`，第 1 维也不等于词表第 1 个 Token，第 2 维也不等于第 2 个 Token。

LM Head 使用一组训练形成的权重，把内部特征重新组合成词表分数：

```text
上下文内部表示
→ 分别与各候选 Token 的输出方向进行匹配
→ 得到每个候选的 Logit
```

## 它怎样处理多个位置

同一个 LM Head 会复用于所有 Token 位置：

```text
位置 1 的 Hidden State → 同一套 LM Head → 位置 1 的词表分数
位置 2 的 Hidden State → 同一套 LM Head → 位置 2 的词表分数
```

参数相同不意味着结果相同，因为不同位置的 Hidden State 不同。

## 它是不是“反向 Embedding”

不能简单说是。

- Embedding：根据 Token ID 查到输入向量；
- LM Head：根据 Hidden State 计算所有词表候选分数。

有些模型会通过 [[02-Weight-Tying是什么|Weight Tying]] 让两者共享同一份权重矩阵，但操作方向、输入和输出仍不同。共享权重不等于两步互为可逆操作。

## LM Head 没有直接做什么

- 不把文本切成 Token；
- 不在不同 Token 位置间交换信息；
- 不把原始分数自动解释为事实置信度；
- 不决定 Temperature、Top-k 或 Top-p；
- 不把 Token ID Decode 成人类文字。

## 理解检查

1. LM Head 为什么输出 `vocab_size` 个数？
2. 同一个 LM Head 为什么可以处理所有位置？
3. Weight Tying 为什么不代表 LM Head 就是 Embedding 的严格逆运算？

下一篇：[[03-Logits是什么|Logits 是什么]]。
