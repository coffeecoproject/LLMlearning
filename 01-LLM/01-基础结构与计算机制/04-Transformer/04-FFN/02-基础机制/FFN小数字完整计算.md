---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[FFN基础机制概览]]"
previous: "[[FFN核心机制与关键名词]]"
next: "[[FFN参数与深入概览]]"
tags: [llm, ffn, worked-example]
---

# FFN 小数字完整计算

> [!summary]
> 用一个二维输入完成“升维 → 非线性 → 降维 → Residual”，可以把 FFN 从抽象名词还原成普通数值变换。

> [!warning] 教学示例
> 下面所有权重、向量和 Token 都是人为设计的极小数字，不来自真实模型。真实 LLM 会使用更大维度、更多 Batch，并常采用 SiLU 或 SwiGLU 等结构。

## 输入

假设某个 Token 位置经过 Attention 和第一次 Residual 后，进入 FFN 分支的简化向量是：

```text
x = [1,2]
```

这里：

```text
hidden_size = 2
intermediate_size = 4
```

## 第一步：升维

教学上假设 `W_up` 学到了四种组合：

```text
中间数1 = 输入1 + 输入2       = 1 + 2 = 3
中间数2 = 输入1 - 输入2       = 1 - 2 = -1
中间数3 = 2 × 输入1           = 2 × 1 = 2
中间数4 = 输入2               = 2
```

于是：

```text
[1,2] → [3,-1,2,2]
```

这就是 `2→4` 的 Up Projection。不是新增 Token，而是产生四个中间特征数值。

## 第二步：非线性

为了直观，使用 ReLU：负数变 0，正数保留。

```text
[3,-1,2,2]
→ ReLU
→ [3,0,2,2]
```

非线性让两次线性投影不能简单合并成一个等价线性矩阵。

## 第三步：降维

教学上假设 `W_down` 使用下面的组合：

```text
输出1 = 0.1 × 中间数1 + 0.1 × 中间数3
      = 0.1 × 3 + 0.1 × 2
      = 0.5

输出2 = 0.1 × 中间数4
      = 0.1 × 2
      = 0.2
```

因此：

```text
[3,0,2,2] → [0.5,0.2]
```

FFN 已经从中间宽度 4 回到主干宽度 2。

## 第四步：Residual 相加

FFN 输出是变化向量：

```text
主干：       [1.0,2.0]
FFN变化：    [0.5,0.2]

相加结果：   [1.5,2.2]
```

`[1.5,2.2]` 才是这次 FFN Residual 更新后的 Hidden State。

## 把四步连起来

```text
[1,2]
→ Up Projection
→ [3,-1,2,2]
→ ReLU
→ [3,0,2,2]
→ Down Projection
→ [0.5,0.2]
→ 加回主干 [1,2]
→ [1.5,2.2]
```

## 这个例子证明了什么

- FFN 处理的是向量；
- `hidden_size` 可以在内部暂时扩展；
- Activation 位于中间；
- Down Projection 恢复主干宽度；
- FFN 输出不是 Token，而是 Residual 更新量。

## 这个例子没有证明什么

- 真实模型一定使用 ReLU；
- 每个中间维度都有一个可读名称；
- FFN 只做很小变化；
- 所有模型都只有两组投影；
- `[1.5,2.2]` 可以被人直接翻译成某个词义。

## 理解检查

1. 为什么升维后仍然只有一个 Token 位置？
2. ReLU 在示例中改变了哪个数？
3. 为什么 Down Projection 要回到 2 维？
4. `[0.5,0.2]` 是最终 Hidden State，还是 FFN 变化？

基础机制到这里已经完成。需要继续理解形状和参数时，进入：[[FFN参数与深入概览|FFN 参数与深入]]。
