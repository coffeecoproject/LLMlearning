---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Token选择策略概览|Token选择策略概览]]"
previous: "[[02-Greedy与Sampling|Greedy与Sampling]]"
next: "[[04-Top-k与Top-p|Top-k与Top-p]]"
tags: [llm, generation, temperature, probability]
---

# Temperature 怎样影响分布

> [!summary]
> Temperature 调整候选之间的相对差距：低温让高分候选更占优势，高温让较低分候选获得更多机会；它不增加模型知识，也不判断事实真假。

## 直觉例子

原本三个候选可能是：

```text
“北京”  0.70
“上海”  0.20
“广州”  0.10
```

低 Temperature 处理后可能更像：

```text
“北京”  0.92
“上海”  0.06
“广州”  0.02
```

高 Temperature 处理后可能更像：

```text
“北京”  0.48
“上海”  0.30
“广州”  0.22
```

这些数字只展示趋势，不是对某个真实模型的实际计算。

## 它怎样工作（可选小公式）

常见做法是在 Softmax 前把 Logits 除以温度 `T`：

```text
调整后 Logit = 原 Logit ÷ T
```

- `T < 1`：分数差被放大，分布更尖；
- `T = 1`：保持原尺度；
- `T > 1`：分数差被压缩，分布更平。

实际软件通常把“完全取最高分”作为 Greedy 单独实现，不建议把 `T = 0` 当作普通除法来理解。

## Temperature 不会做什么

- 不会查询外部事实；
- 不会修改模型权重；
- 不会保证低温回答一定正确；
- 不会直接限制候选数量。

限制候选集合通常由 Top-k、Top-p 或其他过滤规则完成。

## 常见误解

- “高温让模型更聪明”不准确，它主要改变选择多样性。
- “低温消除幻觉”不准确，它只让模型更坚定地沿高分路径生成。
- Temperature 的同一数值在不同模型和任务上不一定有完全相同的行为效果。

## 理解检查

1. 低 Temperature 为什么更偏向最高分候选？
2. Temperature 与 Top-k 的职责有什么区别？
3. 为什么低 Temperature 仍可能稳定地生成错误答案？

下一篇：[[04-Top-k与Top-p|Top-k 与 Top-p]]。
