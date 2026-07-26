---
type: subsection-index
module: 1
status: planned
audience: non-specialist
parent: "[[Transformer概览]]"
previous: "[[FFN概览]]"
next: "[[Output-Layer输出层概览]]"
tags: [llm, transformer-block, hidden-state, layers]
---

# Block 堆叠与 Hidden State 流动

> [!summary]
> 每个 Block 都接收上一层的 Hidden States 并产生同形状的新表示；多层堆叠使各位置逐步形成更丰富的上下文表示。

> [!info] 两阶段共同
> Hidden States 在训练和运行的每次前向计算中都会逐层产生，它们是当前输入对应的临时状态，不是训练后永久保存的新模型参数。训练阶段保存和更新的是产生这些状态的权重；普通运行阶段只重新计算状态。

计划展开：层与 Block 的关系、Hidden State 怎样逐层变化、浅层与深层不能简单贴固定语义标签，以及 Final Hidden State 怎样交给 Output Layer。
