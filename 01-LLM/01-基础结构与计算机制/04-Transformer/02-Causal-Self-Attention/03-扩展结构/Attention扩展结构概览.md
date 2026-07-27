---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Causal-Self-Attention概览]]"
previous: "[[Multi-Head-Attention概览]]"
next: "[[MHA-GQA与MQA概览]]"
tags: [llm, attention, mha, gqa, mqa, mla, optional]
---

# Attention 扩展结构

> [!summary]
> 这一层比较现代模型怎样改变多头 Q/K/V 的组织或压缩方式，以降低 KV Cache 和 Attention 运行成本。

## 阅读顺序

1. [[MHA-GQA与MQA概览|MHA、GQA 与 MQA]]：比较 Query Head 与 KV Head 怎样对应和共享；
2. [[MLA与注意力变体概览|MLA 与注意力变体]]：理解 DeepSeek-V2/V3 的低秩 KV 联合压缩与解耦 RoPE。

## 它们改变什么

```text
MHA / GQA / MQA
→ 主要改变 Query Head 与 KV Head 的数量和共享关系

MLA
→ 主要改变 K、V 表示及缓存所需信息的组织方式
```

这些方案没有取消 Attention 的基础因果链：仍然要形成匹配关系、遵守可见性约束，并汇总信息。

## 阅读边界

模型结构决定需要哪些 K/V 表示；KV Cache 怎样分配显存、分页和调度属于推理运行时。具体闭源 GPT 模型采用哪种内部方案，若官方未公布就标记为未知。

下一篇：[[MHA-GQA与MQA概览|MHA、GQA 与 MQA]]。
