---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Self-Attention的目的与边界概览|Self-Attention的目的与边界概览]]"
previous: "[[01-为什么需要位置之间的信息混合|为什么需要位置之间的信息混合]]"
next: "[[00-QKV投影系统概览|QKV投影系统概览]]"
tags: [llm, attention, self-attention, causal-mask]
---

# Self、Attention 与 Causal 分别是什么

> [!summary]
> `Self` 表示信息来自同一序列，`Attention` 表示相关程度由当前表示动态计算，`Causal` 表示当前位置只能读取自己和过去，不能读取未来。

## 先把三个词拆开

```text
Self
→ Q、K、V 来自同一组序列表示

Attention
→ 每个位置动态决定应从哪些位置汇总多少信息

Causal
→ 通过可见性约束禁止读取未来位置
```

三个词描述不同属性，不能互相替代。

## `Self`：为什么叫自注意力

假设输入序列是：

```text
[我] [喜欢] [苹果]
```

每个位置都从这同一条序列中寻找信息：

```text
“苹果”位置
→ 可以与同一序列中的“我”“喜欢”“苹果”建立关系
```

这叫 Self-Attention。`Self` 不是说每个 Token 只关注自己，而是说查询和被查询的信息来自同一序列。

与之对比，Encoder–Decoder 架构中的 Cross-Attention 可以让 Decoder 的状态去读取 Encoder 输出。两边来源不同，因此称为 Cross-Attention。

## `Attention`：注意力不是意识

Attention 是一套数值计算机制：

```text
计算位置之间的匹配分数
→ 归一化为权重
→ 按权重汇总 Value
```

它借用了“注意力”这个名称，但不代表模型拥有人类意识、主观体验或真正的视觉焦点。

## `Causal`：为什么不能看未来

在自回归 Decoder-only 架构中，位置 `i` 的表示只能依赖位置 `0` 到位置 `i`，不能依赖位置 `i` 后面的内容。这是一条结构性的可见范围规则。

例如：

```text
[今天] [天气] [很] [好]
```

在建立“天气”位置的表示时，它可以读取“今天”和“天气”，但不能读取后面的“很”和“好”。Causal Mask 会限制：

```text
位置 0 → 只能看位置 0
位置 1 → 可以看位置 0、1
位置 2 → 可以看位置 0、1、2
位置 3 → 可以看位置 0、1、2、3
```

形成一个下三角形的可见区域。

> [!note] 阶段边界
> 本节只解释 Attention 内部的可见性结构。为什么训练时必须防止未来信息泄漏，以及模型怎样逐 Token 生成内容，分别留到训练模块和普通运行模块。

## 为什么可以关注自己

当前位置自己的 Token 也是形成新表示的重要信息。如果完全排除自己，汇总结果可能只剩上下文而丢失当前位置内容。

所以典型 Causal Self-Attention 的规则是：

```text
可以看自己和过去
不能看未来
```

## Causal 与位置机制再次区分

```text
RoPE / ALiBi 等位置机制
→ 让模型利用顺序和距离

Causal Mask
→ 决定某个位置是否允许被读取
```

一个过去位置可能很远，但仍然可见；一个未来位置即使只相隔一步，也必须被屏蔽。

## Attention Weight 能解释模型思考吗

Attention Weight 可以显示某一层、某个 Head 在一次计算中怎样分配 Value 汇总权重，但它不是完整的因果解释，因为模型结果还受到：

- 其他 Attention Head；
- 其他模型层；
- FFN / MLP；
- Residual Stream；
- 输入和输出投影；
- 所有训练后的参数。

因此不能看到某个高权重，就直接断言“模型思考时只依据这个词”。

## 常见误解

- **“Self-Attention 就是 Token 只看自己。”** `Self` 指信息来自同一序列。
- **“Attention 说明模型具有人类意识。”** 它是加权汇总的数学机制。
- **“Causal 表示只能看前一个 Token。”** 它通常允许看见全部过去位置和自己。
- **“位置机制已经表达顺序，所以不需要 Causal Mask。”** 一个表达位置，一个限制未来可见性。
- **“Attention Weight 就是模型完整思维链。”** 它只反映整个网络中的一部分计算。

## 理解检查

1. Self-Attention 中的 `Self` 为什么不等于“只关注自己”？
2. 为什么 Decoder-only LLM 必须屏蔽未来 Token？
3. 位置机制与 Causal Mask 分别负责什么？
4. 为什么不能把 Attention Weight 当成完整思考解释？

下一节：[[00-QKV投影系统概览|Q、K、V 投影系统]]。
