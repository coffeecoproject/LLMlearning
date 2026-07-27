---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-MHA-GQA与MQA概览|MHA-GQA与MQA概览]]"
previous: "[[02-MHA怎样一一对应Q与KV-Head|MHA怎样一一对应Q与KV-Head]]"
next: "[[04-GQA怎样让Query-Head分组共享KV|GQA怎样让Query-Head分组共享KV]]"
tags: [llm, mqa, multi-query-attention, key-value-head]
---

# MQA 怎样让所有 Query Head 共享 KV

> [!summary]
> MQA（Multi-Query Attention）保留多个独立 Query Head，但只使用一组 Key Head 和 Value Head；所有 Query Head 都与这组共享 K 比较，并从共享 V 中分别汇总自己的 Context。

## 四个 Query Head 的关系图

```text
Q0 ─┐
Q1 ─┼→ K0、V0
Q2 ─┤
Q3 ─┘
```

与标准 MHA 对比：

```text
标准 MHA：4 个 Query Head + 4 个 KV Head
MQA：     4 个 Query Head + 1 个 KV Head
```

MQA 中的 `Multi-Query` 指多个 Query，不是多个用户请求。

## 共享 K，Weight 为什么还能不同

假设某个接收位置有两个 Query Head：

```text
Q0 = 一组 Query 数值
Q1 = 另一组 Query 数值

两者共同使用 K0
```

计算关系是：

```text
Q0 与 K0 比较 → Score0 → Weight0
Q1 与 K0 比较 → Score1 → Weight1
```

因为 Q0、Q1 不同，即使 K0 相同，Score 和 Softmax Weight 仍可以不同。

## 共享 V，Context 为什么还能不同

两个 Query Head 共用 V0，但它们使用自己的 Weight：

```text
Weight0 × V0 → Context0
Weight1 × V0 → Context1
```

相同的 Value 序列经过不同配比汇总，仍可得到不同 Context。

所以 MQA 共享的是 K/V 表示来源，不是强迫所有 Query Head 输出相同结果。

## 一个极小数字示意

假设共享 V0 中有三个来源值，为了易读暂时把每个 Value 简化成一个数字：

```text
V0 = [10, 20, 30]

Weight0 = [0.7, 0.2, 0.1]
Context0 = 0.7×10 + 0.2×20 + 0.1×30 = 14

Weight1 = [0.1, 0.2, 0.7]
Context1 = 0.1×10 + 0.2×20 + 0.7×30 = 26
```

这组教学数字只说明“共享 V 仍可产生不同汇总结果”，不代表真实模型的 Value 是一维数字。

## MQA 减少了什么，没有减少什么

减少的是：

```text
独立 K Head 数量
独立 V Head 数量
```

仍保留的是：

```text
多个 Query Head
每个 Query Head 自己的 Score、Weight、Context
多个 Context 的合并与 Output Projection
```

## 为什么会设计 MQA

MQA 原始论文把主要问题放在增量解码：反复读取大量 K/V 张量会带来内存带宽成本，而共享 K/V 能显著减少相应张量规模。

> [!note] 阶段边界
> 这句话解释结构动机，不在这里推导 KV Cache 大小或运行速度。训练时同样使用 MQA 结构；真正的 Decode、显存和带宽分析留到普通运行模块。

## 来源与证据边界

[Fast Transformer Decoding: One Write-Head is All You Need](https://arxiv.org/abs/1911.02150) 提出了 Multi-Query Attention，并报告其解码速度提升以及相对基线的小幅质量损失。那是论文实验结果，不应推广成“所有 MQA 模型必然只损失固定质量”。核对日期：2026-07-27。

## 常见误解

- **“MQA 只有一个 Query Head。”** 恰好相反，它保留多个 Query Head，只共享 K/V。
- **“共享 K 后所有 Weight 都一样。”** 不同 Query 仍能产生不同 Score 和 Weight。
- **“共享 V 后所有 Context 都一样。”** 不同 Weight 可以从相同 V 得到不同汇总。
- **“一组 KV 表示整段序列只有一对向量。”** 每个位置仍有对应 K/V，只是 Head 维度共享。
- **“MQA 只在运行时出现。”** 它是模型结构，训练与运行都使用；运行效率是主要设计动机之一。

## 理解检查

1. 8 个 Query Head 的 MQA 有几个 KV Head？
2. 多个 Query Head 使用同一个 K，为什么 Weight 仍可能不同？
3. 多个 Query Head 使用同一个 V，为什么 Context 仍可能不同？
4. MQA 减少的是哪个维度的独立表示？

下一篇：[[04-GQA怎样让Query-Head分组共享KV|GQA怎样让Query-Head分组共享KV]]。
