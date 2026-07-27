---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer输出层概览|Output-Layer输出层概览]]"
previous: "[[05-训练与运行怎样使用Logits|训练与运行怎样使用Logits]]"
next: "[[01-输出形状与LM-Head参数量|输出形状与LM-Head参数量]]"
tags: [llm, output-layer, parameters, shapes]
---

# Output Layer 参数与深入

> [!summary]
> Output Layer 最关键的尺寸关系是 `hidden_size → vocab_size`；LM Head 的参数量主要由这两个数决定，而 Weight Tying 决定它是否与输入 Embedding 共享权重。

## 阅读顺序

1. [[01-输出形状与LM-Head参数量|输出形状与 LM Head 参数量]]；
2. [[02-Weight-Tying是什么|Weight Tying 是什么]]；
3. [[03-Logits到概率的小数字示例|Logits 到概率的小数字示例]]；
4. [[04-真实模型Output-Layer观察|真实模型 Output Layer 观察]]。

## 核心配置联系

```text
hidden_size
→ 每个 Final Hidden State 的宽度
→ LM Head 输入宽度

vocab_size
→ Tokenizer 词表容量
→ LM Head 输出宽度

tie_word_embeddings
→ 输入 Embedding 与 LM Head 是否共享权重
```

## 为什么这部分放在可选层

只理解 LLM 框架时，知道“LM Head 把内部向量变成词表分数”已经足够。参数量、矩阵方向和 Softmax 数字过程是为了之后阅读真实模型配置，不应成为理解主线的门槛。

下一篇：[[01-输出形状与LM-Head参数量|输出形状与 LM Head 参数量]]。
