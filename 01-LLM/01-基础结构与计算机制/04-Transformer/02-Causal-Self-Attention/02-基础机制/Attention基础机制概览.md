---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Causal-Self-Attention概览]]"
previous: "[[Attention框架速览概览]]"
next: "[[Self-Attention的目的与边界概览]]"
tags: [llm, attention, qkv, score, weight, context]
---

# Attention 基础机制

> [!summary]
> 这一层沿着 QKV、Score、Mask、Weight、Value 汇总和多 Head 合并，完整走过一次 Causal Self-Attention。

## 阅读顺序

1. [[Self-Attention的目的与边界概览|目的与边界]]；
2. [[QKV投影系统概览|Q、K、V 投影系统]]；
3. [[从匹配强弱到信息权重概览|从 Score 到 Weight]]；
4. [[Value加权与Context-Mixing概览|Value 加权与 Context Mixing]]；
5. [[Multi-Head-Attention概览|Multi-Head Attention]]。

## 完整路线

```text
Hidden States
→ 产生 Q、K、V
→ Q 与 K 形成 Score
→ Scaling、位置影响与 Causal Mask
→ Softmax 得到 Weight
→ Weight 加权汇总 Value
→ 多个 Head 合并并输出投影
→ Attention 子层输出
```

每个子专题先解释直觉，再提供简单数字或形状。第一次阅读可以跳过矩阵乘法细节，只要能口述这条因果链。

## 不在基础层展开

- MHA、GQA、MQA 的 KV Head 共享差异；
- MLA 的低秩压缩与解耦 RoPE；
- KV Cache、Paged Attention 和请求调度；
- 用 Attention Weight 解释完整推理过程。

前两项进入 [[Attention扩展结构概览|扩展结构]]，运行优化进入后续运行模块。

下一篇：[[Self-Attention的目的与边界概览|Self-Attention 的目的与边界]]。
