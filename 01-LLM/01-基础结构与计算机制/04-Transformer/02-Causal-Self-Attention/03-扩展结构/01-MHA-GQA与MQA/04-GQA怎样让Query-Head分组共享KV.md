---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-MHA-GQA与MQA概览|MHA-GQA与MQA概览]]"
previous: "[[03-MQA怎样让所有Query-Head共享KV|MQA怎样让所有Query-Head共享KV]]"
next: "[[05-MHA-GQA与MQA完整对比|MHA-GQA与MQA完整对比]]"
tags: [llm, gqa, grouped-query-attention, key-value-head]
---

# GQA 怎样让 Query Head 分组共享 KV

> [!summary]
> GQA（Grouped-Query Attention）把多个 Query Head 分成若干组，同一组内共享一个 KV Head；它让 KV Head 数量介于标准 MHA 和 MQA 之间。

## 八个 Query Head、两个 KV Head

假设：

```text
Query Head 数量 = 8
KV Head 数量 = 2
```

一种常见均匀分组是：

```text
第 0 组：Q0、Q1、Q2、Q3 → KV0
第 1 组：Q4、Q5、Q6、Q7 → KV1
```

每个 KV Head 服务 4 个 Query Head：

```text
group_size = 8 ÷ 2 = 4
```

这里只使用整数除法直觉，不要求记公式。

## GQA 为什么位于两者之间

同样有 8 个 Query Head：

```text
标准 MHA：8 个 KV Head
GQA：     例如 2 个 KV Head
MQA：     1 个 KV Head
```

因此可以把三者放在一条共享程度连续线上：

```text
KV 独立更多                                KV 共享更多
标准 MHA ───────────── GQA ───────────── MQA
```

GQA 不是与 MHA、MQA 毫无关系的第三套 Attention，而是使用中间数量 KV Head 的一般化结构。

## 同组 Query Head 会不会得到相同结果

不会被结构强制相同。以 Q0、Q1 共用 KV0 为例：

```text
Q0 × K0 → Weight0 → Weight0 × V0 → Context0
Q1 × K0 → Weight1 → Weight1 × V0 → Context1
```

Q0、Q1 不同，所以 Weight 可以不同；Weight 不同，所以即使 V0 相同，Context 也可以不同。

## “分组”不是把 Token 分组

GQA 分的是 Query Head：

```text
Q0、Q1、Q2、Q3 组成一组
```

不是把文本切成：

```text
前半句一组
后半句一组
```

每个 Query Head 仍处理序列中各接收位置，并遵守 Causal Mask。Head 分组与 Token 位置、句子分段、用户请求分组都不是同一概念。

## 为什么不全部共享或全部独立

标准 MHA 保留最多的独立 KV Head；MQA 把 KV 共享推到一组。GQA 选择中间数量，是在独立表示路径与运行资源之间做结构取舍。

GQA 原始论文报告：在其 uptraining 实验中，GQA 的质量接近 MHA，同时速度可与 MQA 相比。但这是特定模型与实验条件下的结果，不是所有 GQA 配置都自动获得相同质量和速度。

## 数量需要满足什么

常见均匀 GQA 要让 Query Head 能整齐分给 KV Head：

```text
num_attention_heads
能被 num_key_value_heads 整除
```

例如：

```text
32 个 Query Head ÷ 8 个 KV Head
= 每个 KV Head 服务 4 个 Query Head
```

具体模型仍应以其实现和配置校验，不能仅凭名字猜测分组方式。

## 阶段标注

> [!info] 两阶段共同
> GQA 的 Head 分组属于模型结构，训练和运行都会执行。训练阶段学习各 Query 与共享 KV 投影参数；普通运行阶段按固定分组使用它们。KV Cache 收益属于运行效果，不是运行时临时把 MHA 改成 GQA。

## 来源

- [GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints](https://arxiv.org/abs/2305.13245)。
- 核对日期：2026-07-27。

## 常见误解

- **“GQA 的 Group 是 Token 分组。”** 分组对象是 Query Head。
- **“同组 Head 使用相同 K/V，所以结果完全相同。”** Query 和 Weight 仍可不同。
- **“GQA 一定只有两个 KV Head。”** 两个只是示意，真实数量由配置决定。
- **“GQA 是部署时对 MHA 自动做的优化。”** 它通常是模型已确定的架构；论文也讨论从 MHA checkpoint uptrain 的专门方法。
- **“GQA 永远同时达到 MHA 质量和 MQA 速度。”** 论文结果不能无条件外推到所有模型。

## 理解检查

1. 8 个 Query Head、2 个 KV Head 时，每组通常有几个 Query Head？
2. 同组 Query Head 为什么仍能产生不同 Context？
3. GQA 的分组对象为什么不是 Token？
4. GQA 在 MHA 与 MQA 之间改变了什么？

下一篇：[[05-MHA-GQA与MQA完整对比|MHA-GQA与MQA完整对比]]。
