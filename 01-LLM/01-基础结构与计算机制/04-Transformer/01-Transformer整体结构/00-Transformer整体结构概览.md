---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Transformer概览|Transformer概览]]"
previous: "[[01-Transformer框架速览|Transformer框架速览]]"
next: "[[00-Causal-Self-Attention概览|Causal-Self-Attention概览]]"
tags: [llm, transformer, block, decoder-only]
---

# Transformer 整体结构

> [!summary]
> Transformer 是 LLM 用来反复更新整条 Token 序列表示的模型主体；每个 Block 通过 Attention、FFN、Residual 和 Normalization 分工协作，产生下一层 Hidden States。

> [!info] 两阶段共同
> 这套 Block 前向结构在 LLM 训练和运行时都会经过。训练时参数可在反向传播后更新；普通运行时参数固定，Hidden States 仍会随当前输入逐层重新计算。

## 为什么 Embedding 后还需要 Transformer

Embedding 给每个 Token 位置一个初始向量，但它主要来自该 Token 的 ID。仅靠初始向量，模型还不知道“苹果”在当前句子中指水果还是公司，也没有充分组合句子中的条件、指代和关系。

Transformer 的任务是：

```text
接收所有位置的当前表示
→ 让位置之间有条件地交换信息
→ 分别处理每个位置汇集到的特征
→ 保留重要旧信息并形成新表示
```

## 一个 Transformer Block 有什么

先看职责，不记实现细节：

| 组件 | 核心职责 |
|---|---|
| Causal Self-Attention | 让当前位置从允许读取的其他位置汇集信息 |
| FFN / MLP | 对每个位置已经获得的表示做进一步特征变换 |
| Residual Connection | 给原有表示保留直接通路，并叠加子层产生的变化 |
| Normalization | 调整数值尺度，帮助深层数据流保持稳定 |

因此：

```text
Transformer Block ≠ 只有 Attention
```

Attention 负责“位置之间怎样交换”，FFN 负责“每个位置内部怎样继续处理”。

## 组件是怎样嵌套的

Residual 和 Normalization 不是放在整个 Transformer 堆叠之外，而是在每个 Block 中组织 Attention 与 FFN 的数据流：

```text
Transformer 主体
├── Block 1
│   ├── Attention 子层 + 对应 Residual / Norm
│   └── FFN 子层       + 对应 Residual / Norm
├── Block 2
│   ├── Attention 子层 + 对应 Residual / Norm
│   └── FFN 子层       + 对应 Residual / Norm
└── ……
```

概念上可以这样分类：

| 对象 | 位于哪里 | 做什么 |
|---|---|---|
| Attention | Block 内的计算子层 | 在不同 Token 位置之间汇集信息 |
| FFN | Block 内的计算子层 | 处理每个位置当前拥有的特征 |
| Residual | Block 内、围绕子层的连接关系 | 保留主干并叠加子层变化 |
| Normalization | Block 内、位于子层前后之一 | 调整数值尺度 |

Residual 经常在结构图中画成一条弧线或旁路，所以它更像“连接方式”，而不是一个与 Attention 同类的内容变换器。

## 一条简化数据流

教学上可以先读成：

```text
输入 Hidden States
→ Normalization
→ Causal Self-Attention
→ Residual Connection
→ Normalization
→ FFN / MLP
→ Residual Connection
→ 输出 Hidden States
```

这是常见 Pre-Norm 结构的直觉示意。真实模型也可能使用 Post-Norm、并行 Attention/FFN 或其他变体，所以应记住组件职责，而不是把这张图当成所有模型唯一代码顺序。

## 一个简单形状例子

假设序列有 3 个 Token，`hidden_size=4`：

```text
进入 Block：3 个位置 × 每个位置 4 个数 → [3,4]
离开 Block：3 个位置 × 每个位置 4 个数 → [3,4]
```

虽然形状仍然是 `[3,4]`，里面的数字已经变化，每个位置也可能包含更多上下文信息。

```text
形状不变
≠ 表示内容没有变化
```

## 为什么要堆叠很多 Block

一个 Block 只进行一轮信息交换和特征变换。多个 Block 串联后：

```text
初始表示
→ 第 1 层 Hidden States
→ 第 2 层 Hidden States
→ ……
→ Final Hidden States
```

后层接收前层已经处理过的表示，可以继续组合更复杂的上下文关系。但不能简单规定“第 1 层只学语法、第 20 层只学知识”，真实功能通常是分布式并相互重叠的。

## Decoder-only 在这里是什么意思

Transformer 原始架构可以组成不同形式：

```text
Encoder-only
→ 主要使用双向可见的 Encoder Block

Encoder–Decoder
→ Encoder 先处理输入，Decoder 再读取 Encoder 输出并生成

Decoder-only LLM
→ 使用带 Causal 约束的 Transformer Block 处理已有序列
```

GPT、许多现代通用聊天模型采用 Decoder-only 路线。`Decoder-only` 不表示“跳过文本编码”，也不表示“不使用 Tokenizer”；这里的 Decoder 是 Transformer 架构类型，不是 Tokenizer 的 `decode()`。

## 参数分布在哪里

一个 Block 中常见参数分别属于：

```text
Attention → Q、K、V 与输出投影参数
FFN       → 升维、门控或降维参数
Norm      → 与具体 Norm 形式相关的尺度参数
```

Residual Connection 本身主要是“把旧表示与子层结果相加”的数据通路，通常不像线性投影那样拥有一整张权重矩阵。

参数怎样被训练更新不在本节展开。

## 常见误解

- **“Transformer 就是 Attention。”** Attention 只是一个核心子系统。
- **“每经过一层，Token 数量都会减少。”** 常见 Block 前后通常保留相同序列位置数。
- **“形状不变说明什么也没发生。”** 数值和信息内容已经被更新。
- **“Decoder-only 不需要编码用户文字。”** Tokenizer 编码与 Transformer Decoder 是不同层次的概念。
- **“每层都有一个固定、可读的功能名称。”** 真实能力通常分布在多层和多组件中。

## 理解检查

1. Attention 与 FFN 分别解决什么问题？
2. 为什么 `[3,4]` 经过 Block 后仍可保持 `[3,4]`，但内容已经变化？
3. Residual Connection 为什么不能简单理解为一个新的语义模块？
4. Decoder-only 与 Tokenizer Decode 有什么区别？
5. 为什么模型需要堆叠多个 Block？

下一节：[[00-Causal-Self-Attention概览|Causal Self-Attention]]。
