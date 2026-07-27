---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer输出层概览|Output-Layer输出层概览]]"
previous: "[[04-真实模型Output-Layer观察|真实模型Output-Layer观察]]"
next: "[[01-Output-Layer不等于生成策略|Output-Layer不等于生成策略]]"
tags: [llm, output-layer, boundary, review]
---

# Output Layer 边界与复习

> [!summary]
> Output Layer 的稳定职责到“产生词表 Logits”为止；概率调整、Token 选择、循环生成、Decode 和 API 展示属于围绕模型输出运行的后续系统。

## 阅读顺序

1. [[01-Output-Layer不等于生成策略|Output Layer 不等于生成策略]]；
2. [[02-从文本输入到Logits完整串联|从文本输入到 Logits 完整串联]]；
3. [[03-Output-Layer理解检查|Output Layer 理解检查]]。

## 最短边界图

```text
模型主体与 Output Layer
Final Hidden State → Final Norm → LM Head → Logits

运行生成系统
Logits → 调整与选择 → 新 Token ID → 下一轮

Tokenizer
Token ID → Decode → 可显示文本
```

这些环节会在一次生成中连续发生，但“相邻”不等于“属于同一个组件”。分清组件边界，后面学习 Runtime、Agent 和模型 API 时才不会把所有能力都归给 LLM 本体。

下一篇：[[01-Output-Layer不等于生成策略|Output Layer 不等于生成策略]]。
