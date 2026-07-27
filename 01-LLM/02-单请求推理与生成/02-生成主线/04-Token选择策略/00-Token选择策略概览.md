---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-单请求生成主线概览|单请求生成主线概览]]"
previous: "[[03-为什么运行时仍然不能看未来|为什么运行时仍然不能看未来]]"
next: "[[01-Logits处理与Token选择|Logits处理与Token选择]]"
tags: [llm, generation, logits, decoding-strategy, sampling]
---

# Token 选择策略概览

> [!summary]
> 模型给出整套词表 Logits 后，生成控制器可以先调整或筛选候选，再用确定性或随机性规则选出一个 Token。

## 这里从 Output Layer 接手

```text
Output Layer
→ 原始 Logits
→ Logits Processing / Warping
→ 候选分布
→ Greedy 或 Sampling
→ 一个 Token ID
```

## 阅读顺序

1. [[01-Logits处理与Token选择|Logits 处理与 Token 选择]]；
2. [[02-Greedy与Sampling|Greedy 与 Sampling]]；
3. [[03-Temperature怎样影响分布|Temperature 怎样影响分布]]；
4. [[04-Top-k与Top-p|Top-k 与 Top-p]]。

## 先建立两个分类

- **分数处理或候选过滤**：Temperature、Top-k、Top-p、禁止 Token、重复约束等；
- **最终选择方式**：取最高分，或按概率随机抽样。

不同软件的具体处理顺序和参数组合可能不同，不能把某个库的默认值当成所有模型固有属性。

下一篇：[[01-Logits处理与Token选择|Logits 处理与 Token 选择]]。
