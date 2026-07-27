---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer输出层概览|Output-Layer输出层概览]]"
previous: "[[00-Output-Layer框架速览概览|Output-Layer框架速览概览]]"
next: "[[01-Final-Hidden-State怎样进入输出层|Final-Hidden-State怎样进入输出层]]"
tags: [llm, output-layer, lm-head, logits]
---

# Output Layer 基础机制

> [!summary]
> 这一层的核心不是“把向量解码成文字”，而是把每个位置的最终上下文向量投影成覆盖整个词表的候选分数。

## 阅读顺序

1. [[01-Final-Hidden-State怎样进入输出层|Final Hidden State 怎样进入输出层]]；
2. [[02-LM-Head是什么|LM Head 是什么]]；
3. [[03-Logits是什么|Logits 是什么]]；
4. [[04-Softmax怎样把分数变成概率|Softmax 怎样把分数变成概率]]；
5. [[05-训练与运行怎样使用Logits|训练与运行怎样使用 Logits]]。

## 基础数据流

```text
Final Hidden States
→ Final Norm
→ LM Head
→ Logits
→ 根据阶段进入 Loss 或生成策略
```

## 每一步解决什么问题

| 步骤 | 必要性 | 如果没有它 |
|---|---|---|
| Final Hidden State | 汇集前面所有 Block 的上下文处理结果 | 没有可供输出层读取的最终表示 |
| Final Norm | 让进入输出投影的数值尺度更稳定 | 输出投影面对的数值尺度可能更难控制 |
| LM Head | 把内部特征空间连接到词表空间 | 无法给每个候选 Token 分配分数 |
| Logits | 保留候选之间的原始相对强弱 | Loss 或生成策略没有统一输入 |
| Softmax 等后续处理 | 在需要时形成概率分布 | 原始分数不能直接当作总和为 1 的概率 |

## 先划清三个边界

- Final Norm 已在 [[06-Block内Norm与Final-Norm|Block 内 Norm 与 Final Norm]] 详细解释，本节只讲它怎样连接输出层。
- LM Head 产生分数，但不独自决定采样规则。
- Tokenizer Decode 把选出的 Token ID 还原成可显示内容，它不属于 LM Head。

下一篇：[[01-Final-Hidden-State怎样进入输出层|Final Hidden State 怎样进入输出层]]。
