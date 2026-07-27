---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[MHA-GQA与MQA概览]]"
previous: "[[为什么要区分Query-Head与KV-Head]]"
next: "[[MQA怎样让所有Query-Head共享KV]]"
tags: [llm, mha, query-head, key-value-head]
---

# MHA 怎样一一对应 Q 与 KV Head

> [!summary]
> 在本专题所说的标准 MHA 中，每个 Query Head 都配有自己的一组 Key Head 和 Value Head，因此 Query Head 数量与 KV Head 数量相同。

## 先约定这里的 MHA 含义

`MHA` 是 **Multi-Head Attention**。广义资料有时会把多 Query 的各种注意力实现统称为 multi-head attention；为了清楚比较，本专题使用：

```text
标准 MHA
→ Query Head 数量 = KV Head 数量
→ Q、K、V Head 一一对应
```

## 四个 Head 的最小例子

```text
Q0 → K0、V0
Q1 → K1、V1
Q2 → K2、V2
Q3 → K3、V3
```

每条路径都保留自己的：

```text
Q 表示
K 表示
V 表示
Score
Softmax Weight
Context
```

这就是前一个 Multi-Head Attention 专题使用的基线。

## “一一对应”不表示只看一个位置

`Q0 → K0、V0` 表示 Query Head 0 使用 Key/Value Head 0。K0、V0 仍然包含序列中所有允许读取位置的向量：

```text
Q0 的当前位置 Query
→ 与 K0 中多个可见位置比较
→ 得到一组 Weight
→ 汇总 V0 中多个可见位置
```

Head 编号对应和 Token 位置选择是两个不同层次。

## 标准 MHA 保留了什么独立性

四个 Query Head 对应四个 KV Head，表示不同 Head 可以学习不同的：

- Query 投影子空间；
- Key 投影子空间；
- Value 投影子空间；
- 来源 Weight；
- Context 表示。

这给了各 Head 较完整的独立表示路径，但不保证每个 Head 一定形成清晰、无冗余的人类语义分工。

## 最后仍要合并

一一对应不表示四个 Head 是四个独立模型：

```text
Context0、Context1、Context2、Context3
→ Concat
→ Output Projection
→ 一份 Attention 子层输出
```

完整 Score、Weight、Context 和 `W_O` 已在[[Multi-Head-Attention概览|标准 Multi-Head Attention 专题]]讲过，本篇不重复计算。

## 与另外两种结构的关系

如果 Query Head 有 4 个：

```text
标准 MHA：KV Head = 4
GQA：     KV Head 可以是 2 等中间数量
MQA：     KV Head = 1
```

因此标准 MHA 可以看作“没有跨 Query Head 共享 KV”的一端。

## 阶段边界

标准 MHA 在训练和运行时都保持一一对应。它对参数数量、KV Cache 和运行带宽的具体影响不在本篇展开。

## 来源

- [Attention Is All You Need](https://arxiv.org/abs/1706.03762)，Transformer 与 Multi-Head Attention 原始论文。
- 核对日期：2026-07-27。

## 常见误解

- **“Q0 只能关注 Token 位置 0。”** Head 编号与 Token 位置编号没有这种绑定。
- **“一一对应表示每个 Head 最后独立输出答案。”** Context 会被合并为一份子层输出。
- **“标准 MHA 中四个 Head 的 Weight 一定彼此不同。”** 结构允许不同，但不保证实际结果没有重叠。
- **“所有现代 LLM 都保持 Q/K/V Head 数量相同。”** GQA、MQA 和 MLA 等会改变基线。

## 理解检查

1. 四个 Query Head 的标准 MHA 通常有几个 KV Head？
2. `Q2 → KV2` 为什么不表示只读取 Token 位置 2？
3. 标准 MHA 的 Head 最终为什么仍需要 Concat 和 Output Projection？

下一篇：[[MQA怎样让所有Query-Head共享KV]]。
