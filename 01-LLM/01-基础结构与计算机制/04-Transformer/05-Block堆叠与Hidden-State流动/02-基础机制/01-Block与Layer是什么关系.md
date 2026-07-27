---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Block堆叠基础机制概览|Block堆叠基础机制概览]]"
previous: "[[00-Block堆叠基础机制概览|Block堆叠基础机制概览]]"
next: "[[02-Hidden-State怎样逐层流动|Hidden-State怎样逐层流动]]"
tags: [llm, transformer-block, layer]
---

# Block 与 Layer 是什么关系

> [!summary]
> 在许多 LLM 配置和代码中，一个 Transformer Layer 通常就是一个 Transformer Block；但 `Layer` 也可能泛指 Block 内部的线性层或 Normalization，因此必须结合上下文判断。

## 先看整体层级

```text
Transformer 模型主体
├── Transformer Block 1
│   ├── Attention 子层
│   ├── FFN 子层
│   ├── Normalization
│   └── Residual 连接
├── Transformer Block 2
└── ……
```

这里的 Block 是会重复堆叠的完整基本单元。

## 为什么资料经常写 Layer

在模型配置中常见：

```json
{
  "num_hidden_layers": 36
}
```

这通常表示模型主体中堆叠了 36 个 Transformer Blocks，而不是表示整个模型只有 36 次任意数学操作。

代码中也可能命名为：

```text
layers[0]
layers[1]
...
```

其中每个元素是一个完整 Block。

## Layer 为什么容易歧义

`Layer` 是通用术语，也可能出现在：

- Linear Layer：线性投影层；
- LayerNorm：归一化结构；
- Attention Layer：有时指整个 Attention 子层；
- Transformer Layer：通常指完整 Block；
- Hidden Layer：有时泛指模型中间层。

所以看到“32 层模型”时，应确认官方配置里的字段和代码对象，不能把所有带 Layer 的名词当成同一层级。

## Block 内部的子层不等于新的 Block

```text
Attention 子层
+ FFN 子层
+ Residual / Norm 组织
→ 共同构成一个 Block
```

如果模型有 36 个 Blocks，并不表示它只有 18 个 Attention 和 18 个 FFN；常见结构是每个 Block 各包含一套 Attention 和一套 FFN。

## 参数是否共享

现代 Decoder-only LLM 通常让不同 Block 拥有各自的 Attention、FFN 和 Norm 参数：

```text
Block 1 的参数
≠ Block 2 的参数
```

但参数共享是一种可能的架构设计，不能把“不共享”写成所有 Transformer 的绝对定律。判断真实模型应查看具体配置和实现。

## 常见误解

- “Layer 永远等于 Block。”——需要结合术语所在上下文。
- “36 层表示 36 个线性投影。”——通常表示 36 个完整 Transformer Blocks。
- “所有 Block 只是调用同一套参数 36 次。”——许多 LLM 的 Block 参数彼此独立。
- “FFN 是 Block 之外的下一层。”——FFN 通常位于每个 Block 内。

## 理解检查

1. `num_hidden_layers=36` 通常在描述什么？
2. 为什么 LayerNorm 中的 Layer 不能理解成一个完整 Transformer Block？
3. 一个 Block 通常包含哪些主要组件？

下一篇：[[02-Hidden-State怎样逐层流动|Hidden State 怎样逐层流动]]。
