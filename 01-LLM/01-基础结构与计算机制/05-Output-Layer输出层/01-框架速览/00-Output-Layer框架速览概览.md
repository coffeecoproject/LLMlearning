---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer输出层概览|Output-Layer输出层概览]]"
previous: "[[00-Output-Layer输出层概览|Output-Layer输出层概览]]"
next: "[[00-Output-Layer基础机制概览|Output-Layer基础机制概览]]"
tags: [llm, output-layer, overview, beginner]
---

# Output Layer 框架速览

> [!summary]
> Output Layer 是模型内部表示通向词表候选的“出口”：它把一个 Token 位置的最终 Hidden State 转换成词表中每个 Token 的原始分数。

## 先用一句话看懂

```text
Transformer：理解并更新上下文表示
Output Layer：把这种表示翻译成“下一个 Token 可能是谁”的候选分数
```

## 输入、动作与输出

| 环节 | 直白理解 | 常见形状 |
|---|---|---|
| 输入 | 所有 Token 位置的 Final Hidden States | `[batch_size, sequence_length, hidden_size]` |
| Final Norm | 整理最终向量的数值尺度 | 形状不变 |
| LM Head | 为词表中每个 Token 计算一个分数 | `hidden_size → vocab_size` |
| 输出 | 每个位置对应整套词表 Logits | `[batch_size, sequence_length, vocab_size]` |

## 一个不涉及计算的例子

假设教学词表只有四项：

```text
Token：   猫     狗     睡觉    <EOS>
Logit：  2.1    0.4    3.2     -0.7
```

这表示当前上下文下，模型给四个候选分别打了原始分数。`睡觉` 的分数最高，但此时还没有说明系统一定选择它：Temperature、Top-k、Top-p 和采样策略可能继续处理这些分数。

> [!warning] 示例边界
> 上述词表和分数都是教学虚构；真实模型的词表通常很大，Token 也不一定等于一个中文词。

## 为什么不能直接从 Hidden State 得到 Token

Hidden State 的每一维不是“某个 Token 的专属分数”。如果：

```text
hidden_size = 4096
vocab_size  = 151936
```

那么一个 4096 维内部表示仍需经过 LM Head，才能得到 151936 个词表候选分数。两个空间的职责不同：

```text
Hidden State 空间 → 表达当前上下文
Vocabulary 空间  → 为可输出 Token 排名
```

## 训练与运行在哪里分开

同一模型结构都会先算出 Logits：

```text
两阶段共同：Final Hidden States → Final Norm → LM Head → Logits
```

然后：

```text
训练阶段：Logits → 与正确的下一个 Token 对照 → Loss
运行阶段：Logits → 生成策略 → 选出下一个 Token
```

因此，“产生 Logits”不是只在运行时发生；“采样一个 Token”也不是 Output Layer 本身的全部职责。

## 框架层只记六点

1. Output Layer 位于 Transformer 堆叠之后。
2. 它接收的是 Final Hidden States，不是文字和 Token ID。
3. Final Norm 通常先整理最终向量尺度。
4. LM Head 把 `hidden_size` 映射成 `vocab_size` 个分数。
5. Logits 是原始分数，不是概率。
6. 训练和运行会以不同方式使用 Logits。

## 理解检查

1. Output Layer 为什么不能直接省略 LM Head？
2. Logits 的最后一维为什么是 `vocab_size`？
3. “最高 Logit”是否必然等于最终输出 Token？为什么？

继续阅读：[[00-Output-Layer基础机制概览|Output Layer 基础机制]]。
