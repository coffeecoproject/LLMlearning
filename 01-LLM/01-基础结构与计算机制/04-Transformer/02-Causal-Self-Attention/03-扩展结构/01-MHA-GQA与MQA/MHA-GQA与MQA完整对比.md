---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[MHA-GQA与MQA概览]]"
previous: "[[GQA怎样让Query-Head分组共享KV]]"
next: "[[怎样从模型配置判断Head结构]]"
tags: [llm, mha, gqa, mqa, tensor-shape]
---

# MHA、GQA 与 MQA 完整对比

> [!summary]
> 三者使用同一套 Attention 主线，区别集中在 KV Head 数量：标准 MHA 与 Query Head 数量相同，GQA 使用中间数量，MQA 只使用一个。

## 先用同一组数字比较

假设模型有 8 个 Query Head：

| 结构 | Query Head | KV Head | 每个 KV Head 服务多少 Query Head |
|---|---:|---:|---:|
| 标准 MHA | 8 | 8 | 1 |
| GQA 示例 | 8 | 2 | 4 |
| MQA | 8 | 1 | 8 |

关系图：

```text
标准 MHA
Q0→KV0  Q1→KV1  Q2→KV2  Q3→KV3
Q4→KV4  Q5→KV5  Q6→KV6  Q7→KV7

GQA
Q0、Q1、Q2、Q3 → KV0
Q4、Q5、Q6、Q7 → KV1

MQA
Q0、Q1、Q2、Q3、Q4、Q5、Q6、Q7 → KV0
```

## 形状怎样变化

再假设：

```text
sequence_length = 3
head_dim = 4
Query Head 数量 = 8
```

省略 Batch 后：

| 结构 | Q 形状 | K 形状 | V 形状 |
|---|---:|---:|---:|
| 标准 MHA | `[8,3,4]` | `[8,3,4]` | `[8,3,4]` |
| GQA 示例 | `[8,3,4]` | `[2,3,4]` | `[2,3,4]` |
| MQA | `[8,3,4]` | `[1,3,4]` | `[1,3,4]` |

最重要的观察是：

```text
Q Head 数量保持 8
K/V Head 数量逐步减少
```

## Score 和 Weight 数量会一起减少吗

不会按 KV Head 数量直接减少为 8、2、1 套 Weight。每个 Query Head 仍要用自己的 Q 与被分配的 K 比较：

```text
8 个 Query Head
→ 8 套 Query 路径
→ 8 套 Score / Softmax Weight
→ 8 个 Context 路径
```

GQA、MQA 共享的是 K/V Head，不是把多个 Query Head 的 Softmax 强行合并成一个。

## Context 最后怎样组织

每个 Query Head 仍产生宽度为 `head_dim` 的 Context：

```text
8 个 Query Head × 每个 Context 4 维
→ Concat 宽度 32
→ Output Projection
→ Attention 子层输出
```

因此最终 Context 路径数量通常跟 Query Head 数量走，而不是跟 KV Head 数量走。

## 一张职责对比表

| 问题 | 标准 MHA | GQA | MQA |
|---|---|---|---|
| 多个 Query Head | 保留 | 保留 | 保留 |
| K/V 是否共享 | 不跨 Query Head 共享 | 组内共享 | 全部共享 |
| 每个 Query Head 是否独立计算 Weight | 是 | 是 | 是 |
| 是否仍有多个 Context | 是 | 是 | 是 |
| 是否仍需合并与 Output Projection | 是 | 是 | 是 |
| 是否遵守 Causal Mask | 是 | 是 | 是 |

## 参数与运行资源边界

更少的 KV Head 通常意味着更少的 K/V 投影输出和相关参数，也为运行时减少 K/V 缓存规模提供条件。但不能只靠本表直接得出完整速度倍数，因为真实性能还受：

- 序列长度；
- Batch；
- Attention 内核；
- 硬件和数据类型；
- KV Cache 管理；
- 其他模型组件。

这些属于普通运行与性能评估模块。

## 质量边界

结构上共享更多 K/V，会减少独立 KV 表示路径；但“质量一定下降多少”不能从 Head 数量直接算出。训练数据、模型宽度、层数、训练方式和其他组件都会共同影响结果。

论文报告可以作为特定实验事实，不能替代对具体模型的评测。

## 常见误解

- **“KV Head 减少，所以 Query Head 和 Weight 也只剩一个。”** MQA 仍保留多个 Query Head 及各自 Weight。
- **“GQA 的 `[2,3,4]` 表示只有两个 Token。”** 第一维是 KV Head，不是 Token 数量。
- **“MQA 不再做 Multi-Head Attention。”** 它仍有多个 Query Head 和 Context 路径，只是共享 K/V。
- **“KV Head 越少，任何模型都必然越好。”** 资源与表示能力存在取舍，需由具体训练和评测验证。

## 理解检查

1. 为什么 GQA 的 Q 形状可以是 `[8,3,4]`，K/V 却是 `[2,3,4]`？
2. MQA 有 8 个 Query Head 时，通常产生几套 Query Weight？
3. 哪个数量决定 Concat 前通常有多少个 Context 路径？
4. 为什么不能从 KV Head 数量直接推导真实速度倍数？

下一篇：[[怎样从模型配置判断Head结构]]。
