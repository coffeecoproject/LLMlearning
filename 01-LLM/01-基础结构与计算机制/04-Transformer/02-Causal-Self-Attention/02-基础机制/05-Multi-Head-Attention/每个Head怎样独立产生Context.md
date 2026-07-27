---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[Multi-Head-Attention概览]]"
previous: "[[QKV怎样组织成多个Head]]"
next: "[[Head拼接与Output-Projection]]"
tags: [llm, attention-head, attention-weight, context-vector]
---

# 每个 Head 怎样独立产生 Context

> [!summary]
> Q、K、V 按 Head 组织后，每个 Head 都独立完成 Score、Mask、Softmax 和 Value 加权求和，为序列中的每个位置产生自己的 Context。

## 把单头流程复制到每个 Head

假设有两个 Head：

```text
Head 1：Q₁、K₁、V₁
Head 2：Q₂、K₂、V₂
```

接下来分别执行：

```text
Head 1：Q₁K₁ᵀ → Score₁ → Mask → Weight₁ → Context₁
Head 2：Q₂K₂ᵀ → Score₂ → Mask → Weight₂ → Context₂
```

Head 1 不用 Head 2 的 Query 与自己的 Keys 比较；在标准 MHA 中，各 Head 先沿各自路径完成 Attention，结果到后面才合并。

## 两个 Head 为什么会得到不同 Weight

下面所有数字均为教学示意。假设某个接收位置可以读取三个来源位置：

```text
Head 1 Weight：[0.70, 0.20, 0.10]
Head 2 Weight：[0.10, 0.30, 0.60]
```

两组 Weight 不同，是因为两个 Head 的 Query 和 Keys 来自不同投影表示。它们还会分别作用于各自的 Values：

```text
Context₁ = 0.70V₁₀ + 0.20V₁₁ + 0.10V₁₂
Context₂ = 0.10V₂₀ + 0.30V₂₁ + 0.60V₂₂
```

因此两个 Context 不仅来源配比可能不同，所混合的 Value 表示本身也不同。

## Causal Mask 是否每个 Head 都不同

在同一次标准 Causal Self-Attention 中，各 Head 通常遵守相同的基本因果读取权限：当前位置不能读取未来 Token。

```text
Mask：决定允许读谁
Head 内的 Score 和 Softmax：决定允许范围内各来源占多少
```

所以多个 Head 可以有不同 Weight，但不能借此绕过 Causal Mask 读取未来位置。某些模型还会增加局部窗口等结构限制，那属于注意力变体。

## “独立”到底是什么意思

这里的独立是计算路径上的独立：

- 使用自己的 Q/K/V 子表示；
- 计算自己的 Scores；
- 形成自己的 Weights；
- 汇总自己的 Values。

它不表示 Head 是彼此隔绝的独立模型。它们：

- 共享上游输入 Hidden States；
- 在同一个模型目标下共同训练；
- 输出会被拼接和投影；
- 后续层会继续混合它们带来的信息。

## 形状怎样变化

若 `sequence_length=3、num_heads=2、head_dim=4`：

```text
每个 Head 的 Q/K/V： [3,4]
每个 Head 的 Score： [3,3]
每个 Head 的 Weight：[3,3]
每个 Head 的 Context：[3,4]
两个 Head 合在一起： [2,3,4]
```

Weight 中每一行仍对应一个接收位置；Context 中每一行仍是对应位置的新表示。多 Head 没有改变“逐位置产生 Context”的原则。

## 为什么现在还不是最终输出

此刻有多个并列结果：

```text
位置 0：Context₁₀、Context₂₀
位置 1：Context₁₁、Context₂₁
位置 2：Context₁₂、Context₂₂
```

模型还需要对每个位置把这些 Head 结果合并，并通过 Output Projection。否则输出仍分散在多个 Head 维度中，无法直接作为统一的 Attention 子层结果进入后续主通路。

## 常见误解

- **“不同 Head 可以看到不同未来范围。”** 标准 Causal Mask 对所有 Head 保持因果约束。
- **“Head 2 会读取 Head 1 的 Context 再计算。”** 同一层内的 Head 通常并行从相同上游输入出发。
- **“不同 Head 只有 Weight 不同，Value 完全相同。”** 标准 MHA 中各 Head 通常也有自己的 Value 子表示。
- **“独立计算意味着永不交互。”** 它们会在 Concat 和 Output Projection 后重新组合。

## 理解检查

1. 为什么两个 Head 对同一接收位置可能产生不同 Weight？
2. Causal Mask 相同是否意味着各 Head 的 Weight 也相同？
3. “独立计算”为什么不等于“独立模型”？
4. 多个 Head 的 Context 为什么还不能直接叫作统一的子层输出？

下一篇：[[Head拼接与Output-Projection]]。
