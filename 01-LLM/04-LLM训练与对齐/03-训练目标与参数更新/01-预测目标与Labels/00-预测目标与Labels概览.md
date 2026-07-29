---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-训练目标与参数更新概览|训练目标与参数更新概览]]"
next: "[[00-Forward与Loss概览|Forward 与 Loss 概览]]"
tags: [llm, causal-lm, labels, teacher-forcing]
---

# 预测目标与 Labels 概览

> [!summary]
> Decoder-only LLM 的常见预训练目标，是让序列中每个有效位置根据左侧上下文预测下一个 Token；Labels 提供正确对照，Causal Mask 阻止读取未来。

## 主线

```text
完整训练序列
→ 构造当前位置和后一个 Token 的目标关系
→ 使用 Causal Mask 限制可见信息
→ 在多个位置同时产生下一 Token 预测
→ 用 Labels 作为正确对照
```

## 两个核心问题

1. [[01-Causal-Language-Modeling与下一Token目标|Causal Language Modeling 与下一 Token 目标]]
2. [[02-Labels错位与Teacher-Forcing|Labels 错位与 Teacher Forcing]]

## 必须保留的阶段边界

- 训练阶段已经知道完整序列，但未来 Token 只作为外部目标，不作为当前位置的 Attention 输入。
- 训练可以并行计算多个位置的预测，不等于一次运行生成多个相互依赖的未来 Token。
- Teacher Forcing 描述训练时使用真实前缀；普通自回归生成使用模型刚生成的 Token 继续下一轮。
