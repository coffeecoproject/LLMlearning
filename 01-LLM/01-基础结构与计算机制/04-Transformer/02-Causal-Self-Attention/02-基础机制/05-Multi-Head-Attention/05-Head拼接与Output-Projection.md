---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Multi-Head-Attention概览|Multi-Head-Attention概览]]"
previous: "[[04-每个Head怎样独立产生Context|每个Head怎样独立产生Context]]"
next: "[[06-Multi-Head-Attention完整流程|Multi-Head-Attention完整流程]]"
tags: [llm, concat, output-projection, attention-output]
---

# Head 拼接与 Output Projection

> [!summary]
> 各 Head 为同一位置产生 Context 后，模型先沿特征维度把它们拼接起来，再通过可学习的 Output Projection 混合这些特征，得到统一的 Attention 子层输出。

## Concat 怎样合并多个 Head

假设某个位置有两个 Head，每个输出 2 维 Context：

```text
Head 1 Context：[1.0, 0.0]
Head 2 Context：[0.5, 2.0]
```

Concat 的结果是：

```text
[1.0, 0.0] 与 [0.5, 2.0]
→ [1.0, 0.0, 0.5, 2.0]
```

它是把维度依次排列，不是：

```text
求和：[1.5, 2.0]
平均：[0.75, 1.0]
只选择其中一个 Head
```

拼接先完整保留各 Head 的结果，后面的投影再学习怎样组合。

## 为什么拼接后还需要 Output Projection

拼接只完成结构整理，没有让不同 Head 的信息发生可学习组合。Output Projection 使用参数矩阵 `W_O`：

```text
拼接后的表示 × W_O
→ Attention 子层输出
```

它可以学习：

- 怎样混合来自不同 Head 的维度；
- 哪些组合对下一步表示更新更有用；
- 怎样把拼接宽度映射回模型主通路需要的 `hidden_size`。

即使拼接宽度本来就等于 `hidden_size`，`W_O` 仍不只是改形状。它是一层可学习线性变换，可以重新组合所有输入维度。

## 一个形状例子

```text
sequence_length = 3
num_heads = 2
head_dim = 4
hidden_size = 8
```

每个 Head 的结果：

```text
Head 1 Context：[3,4]
Head 2 Context：[3,4]
```

沿最后的特征维拼接：

```text
Concat Context：[3,8]
```

经过 Output Projection：

```text
[3,8] × W_O[8,8] → [3,8]
```

这里的矩阵形状只用于标准 MHA 教学示意，不要求手算 8×8 矩阵。

## Output Projection 与 Softmax 的区别

| 步骤 | 发生位置 | 作用 |
|---|---|---|
| Softmax | 每个 Head 内部 | 把来源 Scores 转成相对 Weights |
| Value 加权求和 | 每个 Head 内部 | 形成各 Head 的 Context |
| Concat | 所有 Head 之后 | 保留并排列多个 Head 结果 |
| Output Projection | Concat 之后 | 用学习参数混合结果并返回主表示空间 |

Output Projection 不负责选择下一个 Token，也不把数值转成概率。下一个 Token 的 Logits 属于模型末端的 Output Layer。

## 这一步之后还缺什么

完成 `W_O` 后，得到的是 Attention 子层的输出，但还不是完整 Transformer Block 的最终 Hidden State：

```text
Multi-Head Attention 输出
→ Residual Connection / Normalization
→ FFN
→ 后续 Residual / Normalization
→ 下一层 Hidden States
```

不同架构可能使用 Pre-Norm 或 Post-Norm，因此具体先后顺序留到 Residual 与 Normalization 专题。

## 参数属于哪里

`W_O` 是 Attention 子层的可学习参数。它与产生 Q、K、V 的投影参数一起，在训练阶段通过误差信号更新；普通运行时只读取这些已经训练好的值完成前向计算。

本篇只说明参数的归属和前向作用，不展开反向传播。

## 常见误解

- **“Concat 就是把多个 Head 相加。”** Concat 保留所有维度，不做逐元素求和。
- **“拼接宽度等于 hidden_size，所以 W_O 没作用。”** 相同形状不代表相同数值，`W_O` 会学习重新组合维度。
- **“W_O 是词表输出层。”** 它是 Attention 内部投影，不是 LM Head。
- **“完成 W_O 就得到了下一个 Token。”** 后面还有 Block 其余结构、多层堆叠和最终 Output Layer。

## 理解检查

1. 两个 2 维 Head Context 拼接后为什么是 4 维，而不是 2 维？
2. Concat 和 Output Projection 分别负责什么？
3. 为什么输入输出形状相同也仍需要 `W_O`？
4. `W_O` 与最终 LM Head 有什么区别？

下一篇：[[06-Multi-Head-Attention完整流程|Multi-Head-Attention完整流程]]。
