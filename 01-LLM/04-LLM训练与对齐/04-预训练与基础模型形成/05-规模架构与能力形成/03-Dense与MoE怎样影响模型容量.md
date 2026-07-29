---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-规模架构与能力形成概览|规模、架构与能力形成概览]]"
previous: "[[02-Scaling-Law应该怎样理解|Scaling Law 应该怎样理解]]"
next: "[[04-能力涌现是真的突然出现吗|能力涌现是真的突然出现吗]]"
tags: [llm, pretraining, dense, moe, model-capacity, active-parameters]
---

# Dense 与 MoE 怎样影响模型容量

> [!summary] 一句话理解
> Dense 模型让每个 Token 经过相同的一组 FFN 参数；MoE 模型准备多组 Expert，再让每个 Token 只经过其中少数几组，因此可以增加总参数容量，而不让每个 Token 都使用全部 Expert 计算。

## 所属阶段

**两阶段共同，但本篇重点是预训练阶段。**

- 设计和预训练时：决定采用 Dense 还是 MoE，并让不同参数获得训练信号；
- 运行时：架构已经固定，Runtime 只是执行 Router 选出的路径。

FFN、Expert、Router 和 Top-k 的内部结构已经在 [[00-FFN扩展结构概览|FFN 扩展结构]] 中讲过。本篇只回答：这种架构选择怎样改变“模型规模”和“能力容量”的理解。

## Dense 模型怎样使用参数

Dense 意为“稠密”。在一个普通 Dense FFN 中，每个 Token 都经过同一套 FFN 参数。

```text
Token A ─┐
Token B ─┼→ 同一个 Dense FFN
Token C ─┘
```

不同 Token 的输入向量不同，所以计算结果不同；但它们使用的是同一组 Weight。

因此对 Dense 模型来说：

```text
总参数增加
→ 每个 Token 通常也要经过更多相关参数计算
```

这不是严格按同一比例增长，因为 Attention、Embedding 和具体形状也会变化，但总参数与单 Token 计算通常联系较直接。

## MoE 模型怎样使用参数

MoE 在部分 FFN 位置准备多个 Expert：

```text
Expert 1
Expert 2
Expert 3
Expert 4
```

Router 根据当前 Token 的 Hidden State 产生路由分数，再选 Top-k Expert。

例如 `Top-k = 1`：

```text
Token A → Expert 2
Token B → Expert 4
Token C → Expert 2
```

模型需要存储所有 Expert，但每个 Token 只执行被选中的一部分。

## 一个小数字例子

假设忽略 Attention 等公共组件，只比较 FFN：

### Dense

```text
1 个 FFN
100 个参数

总参数：100
每 Token 使用：100
```

### MoE

```text
4 个 Expert
每个 100 个参数
Top-k = 1

总 Expert 参数：4 × 100 = 400
每 Token 使用的 Expert 参数：1 × 100 = 100
```

MoE 用相近的单 Token Expert 计算，提供了更大的总 Expert 参数池。

> [!example] 教学示意
> 真实模型还有 Attention、Embedding、Router、共享 Expert 和输出层，不能直接用 `Top-k ÷ Expert 总数` 计算完整模型的激活参数比例。

## 为什么这可能增加模型容量

如果所有 Token 都共用同一个 Dense FFN，大量不同模式需要在同一组参数中共同表示。

MoE 允许不同 Token 倾向使用不同 Expert：

```text
不同输入模式
→ Router 形成不同路由倾向
→ 不同 Expert 获得不同训练信号
→ 总参数池可以容纳更多可复用变换
```

这里不能简单理解成：

```text
Expert 1 专门存中文
Expert 2 专门存数学
```

真实分工通常是训练自动形成的分布式倾向，不一定对应人类能清晰命名的知识类别。

## 预训练时所有 Expert 怎样学到东西

一个 Token 只激活少数 Expert，但一个 Batch 中有许多 Token，整个训练过程又有大量 Batch。

```text
Token A → Expert 2
Token B → Expert 4
Token C → Expert 1
……
```

长期训练中，不同 Expert 会在不同 Token 上被选择并获得 Gradient。

如果 Router 总把 Token 送给少数 Expert，可能出现：

- 某些 Expert 过载；
- 某些 Expert 很少训练；
- 设备间通信不均衡；
- 总参数虽然很多，却没有被有效利用。

因此 MoE 还需要负载均衡与训练稳定性设计。这些属于规模化训练工程，不在本篇展开。

## 总参数与激活参数必须分开

### 总参数

表示完整模型需要保存多少参数，影响：

- 权重文件大小；
- 加载和存储需求；
- 多设备怎样分布 Expert；
- 模型的潜在总容量。

### 每 Token 激活参数

表示处理一个 Token 时实际经过多少参数路径，影响：

- 单 Token 计算量；
- 推理成本和速度；
- 训练时当前 Token 的主要计算路径。

两者不是同一个数字，也不能用其中一个完全替代另一个。

## 开放模型观察

### OpenAI `gpt-oss`

OpenAI 官方模型卡公开：

```text
gpt-oss-120b
→ 116.8B 总参数
→ 约 5.1B 每 Token 激活参数
→ 128 个 Expert，Top-4

gpt-oss-20b
→ 20.9B 总参数
→ 约 3.6B 每 Token 激活参数
→ 32 个 Expert，Top-4
```

这展示了 MoE 的核心取舍：保存较大的总参数池，但每个 Token 只走其中部分 Expert。

### DeepSeek-V3

DeepSeek-V3 官方仓库公开：

```text
DeepSeek-V3 Base
→ 671B 主模型总参数
→ 每 Token 约 37B 激活参数
→ MoE 架构
```

这些数字不能直接与 Dense 模型做简单性能换算，因为训练数据、架构、训练目标和评测条件都不同。

> [!source]
> 来源：[OpenAI gpt-oss 官方模型卡](https://deploymentsafety.openai.com/gpt-oss/full-evaluations)、[DeepSeek-V3 官方仓库](https://github.com/deepseek-ai/DeepSeek-V3)。核对日期：2026-07-29。

## Dense 与 MoE 的核心取舍

| 角度 | Dense | MoE |
|---|---|---|
| 每个 Token | 通常经过同一套 FFN | 经过少数被选 Expert |
| 总参数与激活参数 | 相对接近 | 可以相差很大 |
| 扩大容量 | 往往同时增加单 Token 计算 | 可以主要增加 Expert 总池 |
| 训练难点 | 规模扩大后的计算与内存 | 还增加路由、负载和通信问题 |
| 能力保证 | 无 | 同样无 |

MoE 并不是“免费获得更多能力”。它把一部分困难从每 Token 密集计算，转移到了：

- 更大的权重存储；
- Router 学习；
- Expert 负载均衡；
- 多设备通信；
- 训练和部署复杂度。

## 与 Scaling Law 的关系

看到一个模型“总参数更大”时，需要先问：

```text
它是 Dense 还是 MoE？
总参数是多少？
每 Token 激活参数是多少？
训练 Token 和计算是多少？
```

不同架构下，单看总参数可能无法准确代表训练计算或运行计算。Scaling Law 实验也需要明确自己采用什么模型规模和计算口径。

## 常见误解

### MoE 有 100 个 Expert，就等于同时运行 100 个模型

不是。Expert 是同一个 Transformer 内部的条件计算组件，共享 Attention、Embedding 和整体训练目标。

### 未被当前 Token 激活的 Expert 不占内存

不是。它们仍然属于模型权重，需要存储或分布在设备上。

### 激活参数少，所以速度一定按比例提高

不一定。Router、内存访问、设备通信、Batch 和硬件利用率都会影响真实速度。

### MoE 总参数更大，所以一定比 Dense 更聪明

不能这样判断。能力仍由数据、训练计算、优化质量、架构细节和后训练共同决定。

## 理解检查

1. Dense 与 MoE 在每个 Token 使用 FFN 参数的方式上有什么不同？
2. 为什么 MoE 的总参数和激活参数必须分别报告？
3. 为什么增加 Expert 数量不等于免费增加能力？

## 继续学习

- 上一篇：[[02-Scaling-Law应该怎样理解|Scaling Law 应该怎样理解]]
- 返回：[[00-规模架构与能力形成概览|规模、架构与能力形成概览]]
- 复习结构：[[00-FFN扩展结构概览|FFN 扩展结构概览]]
- 下一篇：[[04-能力涌现是真的突然出现吗|能力涌现是真的突然出现吗]]
