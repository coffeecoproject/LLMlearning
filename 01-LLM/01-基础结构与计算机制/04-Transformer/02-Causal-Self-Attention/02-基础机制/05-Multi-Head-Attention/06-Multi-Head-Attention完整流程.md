---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Multi-Head-Attention概览|Multi-Head-Attention概览]]"
previous: "[[05-Head拼接与Output-Projection|Head拼接与Output-Projection]]"
next: "[[00-MHA-GQA与MQA概览|MHA-GQA与MQA概览]]"
tags: [llm, multi-head-attention, tensor-shape, forward-pass]
---

# Multi-Head Attention 完整流程

> [!summary]
> Multi-Head Attention 的完整前向链是：输入 Hidden States 经过 Q/K/V 投影并组织成多个 Head，各 Head 独立计算 Context，随后拼接并经过 Output Projection，最终为每个位置产生统一的 Attention 子层输出。

## 教学条件

以下数字和形状是教学示意，不代表某个真实模型配置：

```text
sequence_length = 3
hidden_size = 8
num_heads = 2
head_dim = 4
```

省略 Batch 维度，输入是：

```text
H：[3,8]
```

意思是序列有 3 个位置，每个位置当前用 8 个数字表示。

## 第一步：产生 Q、K、V

```text
H × W_Q → Q_all [3,8]
H × W_K → K_all [3,8]
H × W_V → V_all [3,8]
```

Q、K、V 分别承担查询、匹配索引和可传递内容的角色。三个投影结果数值不同，即使形状相同也不能互换。

## 第二步：组织 Head 维度

因为：

```text
num_heads × head_dim = 2 × 4 = 8
```

所以把每个投影结果从：

```text
[3,8]
```

组织成：

```text
[2,3,4]
```

现在每个 Head 都有 `[3,4]` 的 Q、K、V。

## 第三步：每个 Head 计算 Score

每个 Head 中，三个 Query 位置分别与三个 Key 位置比较：

```text
Head 1 Score：[3,3]
Head 2 Score：[3,3]
```

Score 矩阵的行对应接收位置，列对应来源位置。

## 第四步：Scaling 与 Causal Mask

每个 Head 的 Scores 会经过必要的数值缩放。Causal Mask 禁止当前位置读取未来位置：

```text
位置 0：只能读取位置 0
位置 1：可以读取位置 0、1
位置 2：可以读取位置 0、1、2
```

多个 Head 可以形成不同 Score，但都不能绕过相同的基本因果权限。

位置机制不一定都在这一步加入。例如，[[05-RoPE的作用位置与直觉|RoPE]] 通常先改变 Q、K，再由它们计算 Score；[[06-ALiBi的作用位置与直觉|ALiBi]] 则把与相对距离有关的偏置加入 Score。这里不把不同位置方案强行画成同一条计算步骤，具体路径返回[[00-Position位置机制概览|Position 位置机制]]复习。

## 第五步：Softmax 得到 Weight

每个 Head 独立把允许范围内的 Scores 转成相对份额：

```text
Head 1 Weight：[3,3]
Head 2 Weight：[3,3]
```

每一行允许位置的 Weight 合计为 1，被 Mask 的未来位置贡献为 0。

## 第六步：分别汇总 Value

```text
Weight₁ × V₁ → Context₁ [3,4]
Weight₂ × V₂ → Context₂ [3,4]
```

每个 Head 为三个接收位置分别产生自己的 Context Vector。

## 第七步：拼接各 Head

对每个位置沿特征维拼接：

```text
Context₁ [3,4]
Context₂ [3,4]
→ Concat [3,8]
```

这里保留了两个 Head 的输出，没有平均或只选择一个 Head。

## 第八步：Output Projection

```text
Concat [3,8] × W_O[8,8]
→ Attention Output [3,8]
```

`W_O` 学习怎样混合不同 Head 的结果，并让输出回到模型主通路使用的 8 维表示。

> [!important] 多 Head，仍然只有一个子层输出
> 两个 Head 不是两个候选答案，也不会各自生成 Token。它们为每个位置提供两个 Context，经过 Concat 和 `W_O` 后，仍得到该位置的一份统一 Attention Output；之后才进入 Transformer Block 的其他结构。

## 一张形状总表

| 阶段 | 教学形状 | 含义 |
|---|---:|---|
| 输入 Hidden States | `[3,8]` | 3 个位置，每个位置 8 维 |
| Q/K/V 总投影 | 各 `[3,8]` | 包含两个 Head 的表示 |
| 分 Head 后 Q/K/V | 各 `[2,3,4]` | 2 个 Head，每个位置 4 维 |
| 每个 Head 的 Score | `[3,3]` | 接收位置与来源位置两两匹配 |
| 每个 Head 的 Weight | `[3,3]` | 每个接收位置的一组来源配比 |
| 每个 Head 的 Context | `[3,4]` | 单头逐位置汇总结果 |
| Concat | `[3,8]` | 拼接两个 Head |
| Output Projection | `[3,8]` | 统一的 Attention 子层输出 |

## 可选公式

不要求记忆，公式只压缩已经走过的步骤：

```text
Headᵢ = Attention(Qᵢ, Kᵢ, Vᵢ)

MHA(H) = Concat(Head₁, Head₂, …, Headₙ) × W_O
```

第一行表示每个 Head 独立产生 Context，第二行表示拼接后经过 Output Projection。

## 整体边界

此时得到：

```text
每个 Token 位置的统一 Attention 子层输出
```

尚未得到：

- 完整 Transformer Block 输出；
- 下一层最终 Hidden State；
- 整个模型的最终 Hidden States；
- 词表 Logits；
- 下一个 Token 概率或采样结果。

下一步需要学习 MHA、GQA、MQA 的 K/V Head 共享差异，然后才能回到 Transformer Block 的 Residual、Normalization 和 FFN。

## 常见误解

- **“多 Head 会把序列复制成多份文本。”** 复制和重组的是数值表示路径，不是原始文字。
- **“Score 只有一张矩阵，所有 Head 共用。”** 标准 MHA 中每个 Head 会形成自己的 Score 和 Weight。
- **“Concat 后已经完成学习型混合。”** Concat 只排列维度，`W_O` 才执行可学习组合。
- **“Attention Output 就是最终预测。”** 它还要经过 Block 其余结构、多层堆叠和 Output Layer。

## 理解检查

1. `[3,8]` 为什么能组织成 `[2,3,4]`？
2. 每个 Head 的 Score 为什么是 `[3,3]`？
3. 两个 `[3,4]` 的 Context 怎样变回 `[3,8]`？
4. 哪一步产生来源概率份额，哪一步混合多个 Head？
5. Multi-Head Attention 完成后为什么还不能直接得到下一个 Token？

返回：[[00-Multi-Head-Attention概览|Multi-Head Attention]]。下一专题：[[00-MHA-GQA与MQA概览|MHA、GQA 与 MQA]]。
