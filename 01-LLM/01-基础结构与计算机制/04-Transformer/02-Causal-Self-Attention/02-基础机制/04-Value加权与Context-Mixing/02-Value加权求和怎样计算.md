---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Value加权与Context-Mixing概览|Value加权与Context-Mixing概览]]"
previous: "[[01-为什么Weight要作用于Value|为什么Weight要作用于Value]]"
next: "[[03-Context-Vector是什么|Context-Vector是什么]]"
tags: [llm, value, weighted-sum, vector]
---

# Value 加权求和怎样计算

> [!summary]
> 对一个接收位置，模型把每个来源位置的 Value 向量乘以对应 Weight，再把所有结果逐维相加；输出仍是一个向量。

## 先从一个数字开始

下面是**教学示意**。假设两个来源提供的 Value 暂时都只有一个数字：

```text
Weight = [0.25, 0.75]
Value  = [10, 30]
```

加权求和为：

```text
0.25 × 10 + 0.75 × 30
= 2.5 + 22.5
= 25
```

结果更靠近 `30`，因为第二个来源的 Weight 更大。但真实模型中的 Value 不是一个数字，而是多维向量。

## 两维向量的完整例子

仍然使用虚构数字：

```text
w₀ = 0.25    V₀ = [2, 0]
w₁ = 0.75    V₁ = [0, 4]
```

第一步，Weight 缩放整个 Value 向量：

```text
0.25 × [2, 0] = [0.5, 0.0]
0.75 × [0, 4] = [0.0, 3.0]
```

第二步，对应维度相加：

```text
[0.5, 0.0] + [0.0, 3.0]
= [0.5, 3.0]
```

结果 `[0.5, 3.0]` 是一个新向量，不再只等于 `V₀` 或 `V₁`。

## Weight 乘的是整个向量

一个来源位置只有一个 Weight，但这个 Weight 会缩放该 Value 的所有维度：

```text
0.2 × [3, -1, 5]
= [0.6, -0.2, 1.0]
```

Attention Weight 不是“第一维 0.2、第二维 0.7”这种逐特征权重。它是在当前 Head 中，针对一个接收位置与一个来源位置的系数。

## 三个来源的例子

```text
Weight = [0.2, 0.7, 0.1]

V₀ = [1, 0]
V₁ = [0, 2]
V₂ = [1, 1]
```

逐项缩放：

```text
0.2V₀ = [0.2, 0.0]
0.7V₁ = [0.0, 1.4]
0.1V₂ = [0.1, 0.1]
```

逐维求和：

```text
[0.2 + 0.0 + 0.1, 0.0 + 1.4 + 0.1]
= [0.3, 1.5]
```

这就是一个接收位置的一次 Value 加权求和。

## 为什么不是只取最大 Weight

如果只选最大项，上例只会保留 `V₁`。标准 Softmax Attention 则让多个可见来源都能以不同程度参与：

```text
不是：选择一个来源，丢掉其余来源
而是：按系数组合所有未被 Mask 的来源
```

这使模型可以同时保留主导信息和次要修饰信息。被 Causal Mask 禁止的位置 Weight 为 `0`，因此对应 Value 的贡献也严格为零。

## 可选技术表示

对接收位置 `i`，常写成：

```text
context_i = Σ_j weight_(i,j) × value_j
```

只需这样读：

```text
固定接收位置 i
→ 遍历每个允许的来源位置 j
→ 用 i 对 j 的 Weight 乘 j 的 Value
→ 全部相加
```

若序列长度是 `S`、一个 Head 的 Value 维度是 `d_v`：

```text
所有 Weight： [S, S]
所有 Value：  [S, d_v]
所有结果：    [S, d_v]
```

矩阵写法常简写为：

```text
Context = Weight × V
```

这里不要求掌握矩阵乘法；它只是把“每个接收位置分别做一次加权求和”合并表达。

## 常见误解

- **“加权求和后只剩一个数字。”** Value 是向量，所以结果仍是向量。
- **“Weight 分别控制 Value 的不同维度。”** 一个位置对一个来源的 Weight 会缩放整个 Value 向量。
- **“只需要最高 Weight 的 Value。”** 标准 Attention 通常汇总所有可见来源。
- **“Weight 为 0.7 表示 70% 的最终答案来自该 Token。”** 它只是在当前层、当前 Head、当前接收位置的局部加权系数。

## 理解检查

1. `0.4 × [2, 5]` 等于什么？
2. 权重 `[0.25, 0.75]` 与 Value `[2, 0]`、`[0, 4]` 的加权和是什么？
3. 为什么加权和的结果仍然是一个向量？

下一篇：[[03-Context-Vector是什么|Context Vector 是什么]]。
