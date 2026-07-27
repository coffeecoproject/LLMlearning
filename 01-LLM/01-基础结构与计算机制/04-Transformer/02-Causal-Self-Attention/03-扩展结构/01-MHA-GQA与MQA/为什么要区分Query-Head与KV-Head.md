---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[MHA-GQA与MQA概览]]"
previous: "[[MHA-GQA与MQA概览]]"
next: "[[MHA怎样一一对应Q与KV-Head]]"
tags: [llm, query-head, key-value-head, attention]
---

# 为什么要区分 Query Head 与 KV Head

> [!summary]
> Query Head 决定“以哪种方式发起匹配”，KV Head 提供“用什么 Key 被匹配、匹配后传递什么 Value”；标准 MHA 让它们数量相同，而 GQA、MQA 让多个 Query Head 共享较少的 KV Head。

## 为什么以前可以统称 Head

学习标准 Multi-Head Attention 时，一个 Head 可以完整写成：

```text
Q Head
+ 对应 K Head
+ 对应 V Head
→ Score、Weight、Context
```

因为标准 MHA 中 Q、K、V Head 通常一一对应，所以把三者合称为“一个 Attention Head”不会立即出错。

但到了 GQA 和 MQA：

```text
Query Head 数量 ≠ KV Head 数量
```

如果继续只说“有几个 Head”，就无法判断是在说 Query Head，还是 Key/Value Head。

## Query Head 是什么

每个 Query Head 拥有自己的 Query 子表示。对于某个接收位置，它负责：

```text
当前 Hidden State
→ 产生该 Query Head 的 Q
→ 与它被分配到的 K Head 比较
→ 得到该 Query Head 自己的 Score 和 Weight
```

因此多个 Query Head 即使使用相同 K，也可能因为 Q 不同而产生不同匹配结果。

## KV Head 是什么

这里把一组 Key Head 和对应 Value Head 合称为一个 **KV Head**：

```text
K Head
→ 提供各来源位置用于匹配的 Key

V Head
→ 提供各来源位置最终被加权汇总的 Value
```

说“两个 Query Head 共享一个 KV Head”，表示它们使用相同的一组 K 表示和 V 表示，但各自仍使用不同 Q 发起匹配。

## 一个四对二的示意

假设：

```text
Query Head：Q0、Q1、Q2、Q3
KV Head：   KV0、KV1
```

可以这样分组：

```text
Q0、Q1 → KV0
Q2、Q3 → KV1
```

于是：

```text
Q0 与 K0 比较 → Weight0 → 汇总 V0
Q1 与 K0 比较 → Weight1 → 汇总 V0

Q2 与 K1 比较 → Weight2 → 汇总 V1
Q3 与 K1 比较 → Weight3 → 汇总 V1
```

虽然 Q0、Q1 共用 K0、V0，但 `Weight0` 与 `Weight1` 不必相同，因为参与比较的 Query 不同。

## “共享”发生在哪个维度

共享发生在 **Head 维度**，不是 Token 位置维度。

假设序列有三个位置：

```text
位置 0、位置 1、位置 2
```

一个 KV Head 仍然为三个位置分别提供 K、V：

```text
K：[3, head_dim]
V：[3, head_dim]
```

它不是整句话只有一个 K 向量、一个 V 向量。多个 Query Head 共享的是这套“逐位置 K/V 序列”。

## 与阶段的关系

> [!info] 两阶段共同
> Query Head 和 KV Head 的数量是模型结构。训练时模型在这种共享关系下学习参数，运行时继续使用同样的对应关系；它不是 Runtime 临时决定把哪些 Head 合并。

## 常见误解

- **“KV Head 是一个 Token。”** 它是一条为整段序列逐位置产生 K/V 的表示路径。
- **“共享 K/V 后，所有 Query Head 的 Weight 都相同。”** Q 不同仍可产生不同 Score 和 Weight。
- **“Query Head 自己不再需要 Value。”** 它使用被分配的共享 Value Head形成自己的 Context。
- **“配置写 Head 数量时永远指同一种 Head。”** 必须结合字段名判断 Query Head 还是 KV Head。

## 理解检查

1. 为什么标准 MHA 中可以暂时把 Q/K/V 合称为一个 Head？
2. 两个 Query Head 共用一个 K Head，为什么仍可能得到不同 Weight？
3. “共享发生在 Head 维度”是什么意思？
4. 一个 KV Head 是否表示整段文本只有一个 K 和一个 V？

下一篇：[[MHA怎样一一对应Q与KV-Head]]。
