---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[Value加权与Context-Mixing概览]]"
previous: "[[Context-Vector是什么]]"
next: "[[Multi-Head-Attention概览]]"
tags: [llm, attention, context-mixing, causal-mask]
---

# 为什么每个位置得到不同的 Context

> [!summary]
> 每个接收位置有自己的 Query、自己的可见范围和自己的一组 Attention Weights，因此即使面对同一批来源 Value，各位置通常也会得到不同的 Context Vector。

## 不是全句共用一次混合

假设序列是：

```text
位置 0：小明
位置 1：把
位置 2：苹果
位置 3：给了
位置 4：小红
```

Self-Attention 不是先为整句话算一个共同摘要，再发给所有位置。它会站在每个接收位置的角度分别提问：

```text
位置 0 当前需要从哪里取信息？
位置 1 当前需要从哪里取信息？
位置 2 当前需要从哪里取信息？
……
```

每个位置都对应 Attention Weight 矩阵中的一行，也就对应一次独立的 Value 加权求和。

## 一个三位置教学示意

下面的 Weight 与 Value 都是虚构数字。

三个来源 Value：

```text
V₀ = [1, 0]
V₁ = [0, 2]
V₂ = [1, 1]
```

在 Causal Mask 下，各接收位置可能得到：

```text
接收位置 0：Weight = [1.0, 0.0, 0.0]
接收位置 1：Weight = [0.4, 0.6, 0.0]
接收位置 2：Weight = [0.2, 0.3, 0.5]
```

分别求和：

```text
Context₀ = 1.0V₀                         = [1.0, 0.0]
Context₁ = 0.4V₀ + 0.6V₁                = [0.4, 1.2]
Context₂ = 0.2V₀ + 0.3V₁ + 0.5V₂        = [0.7, 1.1]
```

同一组来源 Value 被不同 Weight 组合，所以三个接收位置得到不同结果。

## 差异来自哪里

### 1. Query 不同

每个位置的输入 Hidden State 通常不同，投影得到的 Query 也不同。不同 Query 与同一组 Keys 比较，会产生不同 Scores 和 Weights。

### 2. Causal 可见范围不同

在 Decoder-only LLM 的 Causal Self-Attention 中：

```text
较早位置：只能读取更少的过去与自身位置
较晚位置：可以读取更多已经出现的位置
```

未来位置被 Mask 后 Weight 为 0，因此不可能参与当前 Context。

### 3. 所在层和 Head 不同

即使接收位置相同，不同层收到的 Hidden State 已经不同；不同 Head 也使用自己的投影或头部子空间。因此 Context Mixing 是层、Head 和位置共同决定的局部结果。

## 同一个 Value 可以贡献给多个位置吗

可以。同一个来源位置的 Value 可以被多个接收位置读取，但获得的 Weight 不一定相同：

```text
接收位置 A 对 V₁ 的 Weight = 0.2
接收位置 B 对 V₁ 的 Weight = 0.7
```

这不表示 Value 自己发生了选择，而是两个接收位置的 Query 与该来源 Key 的匹配关系不同。

## 可选形状直觉

若一条序列长度为 `S`：

```text
Weight： [S, S]
Value：  [S, d_v]
Context：[S, d_v]
```

可以把 Weight 的每一行理解成一个接收位置的“来源配比”。这一行与所有 Value 做一次加权求和，得到 Context 中对应的一行。

## 这一步之后发生什么

到这里，我们只完成了单个 Head 的逐位置 Context Mixing：

```text
每个接收位置的一组 Weights
× 所有可见来源 Values
→ 每个接收位置的单头 Context Vector
```

现代 Transformer 通常同时使用多个 Head。下一节才讨论不同 Head 的结果怎样拼接并通过 Output Projection。不要在本节提前把单头结果叫作完整 Attention 子层输出。

## 常见误解

- **“Attention 为整句话只生成一个摘要。”** 它通常为每个接收位置分别生成结果。
- **“所有位置看到相同来源，所以结果相同。”** Query、Weights 和 Causal 可见范围都可能不同。
- **“同一个 Value 只能被读取一次。”** 它可以向多个允许读取它的接收位置贡献信息。
- **“越靠后的 Token 一定更重要。”** 靠后只意味着可见历史更多，不等于 Weight 必然更高或语义更重要。

## 理解检查

1. 为什么 Weight 矩阵的每一行可以对应一个接收位置？
2. 同一个来源 Value 为什么能对两个接收位置产生不同贡献？
3. Causal Mask 怎样导致较早与较晚位置的可见范围不同？
4. 本节的输出为什么还不能叫作完整的多头 Attention 输出？

下一节：[[Multi-Head-Attention概览|Multi-Head Attention]]。
