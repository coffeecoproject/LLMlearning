---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Block堆叠与Hidden-State流动概览]]"
previous: "[[Block堆叠框架速览概览]]"
next: "[[Block与Layer是什么关系]]"
tags: [llm, transformer-block, hidden-state, mechanism]
---

# Block 堆叠基础机制

> [!summary]
> 这一层把单个 Block 的输入输出、多个 Block 的串联和 Final Hidden States 连成一条完整数据流。

## 阅读顺序

1. [[Block与Layer是什么关系|Block 与 Layer 是什么关系]]；
2. [[Hidden-State怎样逐层流动|Hidden State 怎样逐层流动]]；
3. [[为什么要堆叠多个Block|为什么要堆叠多个 Block]]；
4. [[为什么主干形状通常保持不变|为什么主干形状通常保持不变]]；
5. [[Final-Hidden-State怎样交给输出层|Final Hidden State 怎样交给输出层]]。

## 基础路线

```text
Embedding 产生 H⁰
→ Block 1：H⁰ → H¹
→ Block 2：H¹ → H²
→ ……
→ Block N：Hᴺ⁻¹ → Hᴺ
→ Final Norm
→ Output Layer
```

第一次阅读不需要记层数配置或参数量，只需理解：上一层输出就是下一层输入，同一批 Token 位置的表示被反复更新。

下一篇：[[Block与Layer是什么关系|Block 与 Layer 是什么关系]]。
