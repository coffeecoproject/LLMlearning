---
type: subsection-index
module: 1
status: planned
audience: non-specialist
parent: "[[Causal-Self-Attention概览]]"
previous: "[[Multi-Head-Attention概览]]"
next: "[[MLA与注意力变体概览]]"
tags: [llm, mha, gqa, mqa, attention-variants]
---

# MHA、GQA 与 MQA

> MHA、GQA 和 MQA 的核心结构差别，是多个 Query Head 分别使用独立的 Key/Value Head，还是共享部分或全部 Key/Value Head。

计划结构：标准 MHA、MQA、GQA，以及三者的 Head 对应关系和结构边界对比。

> [!note] 阶段边界
> 本节只解释 Head 怎样对应和共享。它们对 KV Cache、显存占用和推理效率的影响留到普通运行模块。
