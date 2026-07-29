---
type: concept
module: 1
status: complete
audience: non-specialist
reading: optional
parent: "[[00-训练进度与状态概览|训练进度与状态概览]]"
previous: "[[../03-Backward与参数更新/02-Optimizer与Learning-Rate怎样更新参数|Optimizer 与 Learning Rate 怎样更新参数]]"
next: "[[03-Checkpoint保存什么|Checkpoint 保存什么]]"
tags: [llm, training, batch, step, epoch, token-budget]
---

# Batch、Step、Epoch 与 Token Budget 怎样区分

> [!summary] 一句话理解
> Batch 表示一次参与计算的数据集合，Step 通常表示一次参数更新，Epoch 表示数据集被完整遍历一次，而 Token Budget 表示训练计划总共处理多少 Token。

## 所属阶段

**训练阶段。** 运行服务也会使用 Batch 一词，但那是为了把推理请求一起计算，不等于这里的训练 Batch。

## 为什么这些词容易混淆

它们都在描述“训练进行到哪里”，但计量对象不同：

| 名词 | 主要在数什么 | 核心问题 |
|---|---|---|
| Batch | 本次参与计算的数据 | 这次看了哪些样本？ |
| Step | 参数更新次数 | 模型参数改了几次？ |
| Epoch | 完整遍历数据集的次数 | 整套数据看了几遍？ |
| Token Budget | 处理的 Token 总量 | 总共读了多少训练 Token？ |

## Batch 是什么

训练不会只用一个句子，也通常不能一次装入全部数据。系统把若干训练样本组织成一批进行 Forward 和 Backward，这一批就是 Batch。

例如一个示意 Batch：

```text
样本 1：我 喜欢 苹果
样本 2：天空 是 蓝色 的
样本 3：The cat sleeps
```

不同样本长度不一致时，系统还会使用 Padding、分桶或打包等方式组织张量。那是数据装配问题，不改变 Batch 的基本含义。

## Micro-batch 与 Effective Batch

显存一次真正容纳并完成 Forward/Backward 的小批数据，常称为 Micro-batch。

如果系统连续处理多个 Micro-batch、累积梯度后才更新一次参数，那么一次参数更新实际综合的数据量称为 Effective Batch（有效 Batch）。

示例：

```text
每个 Micro-batch：2 个样本
累计次数：4
一次更新覆盖：2 × 4 = 8 个样本
```

如果还使用多个训练设备，有效 Batch 还需要考虑设备数量。

## Step 是什么

不同工具偶尔会把“处理一个 Batch”也称为 step，因此阅读日志时要看具体定义。

在这套学习资料中，若无额外说明：

> 一个 Training Step 指一次 Optimizer Step，也就是一次真正的参数更新。

因为存在梯度累积，一次 Training Step 之前可能已经执行了多次 Forward 和 Backward。

## Epoch 是什么

Epoch 表示训练程序把定义好的有限数据集完整遍历一遍。

例如数据集有 1,000 个样本，有效 Batch 为 100 个样本，那么在忽略边界情况时：

```text
大约 10 个 Step ≈ 1 个 Epoch
```

如果再遍历一次，就是第 2 个 Epoch。

## 为什么大模型预训练常说 Token 而不只说 Epoch

大规模预训练的数据可能：

- 来自多个数据源并按比例混合；
- 经过过滤、去重和重复采样；
- 以流式方式读取；
- 很难把“完整看完一次”定义得足够稳定。

因此，预训练常用处理过的 Token 数或训练 Step 数描述规模，例如“训练使用了多少万亿 Token”。

Token Budget 是训练计划允许模型处理的 Token 总量。它比“有多少篇文档”更接近模型实际收到的训练信号数量，但也不能单独说明数据质量。

## 一个串联例子

假设：

- 每个 Micro-batch 含 2 个序列；
- 每个序列整理后有 1,000 个有效训练 Token；
- 累积 4 个 Micro-batch 后更新；
- 暂时只用一台设备。

那么一次 Optimizer Step 大致覆盖：

```text
2 × 1,000 × 4 = 8,000 个训练 Token
```

经过 100 个这样的 Step，大致处理：

```text
8,000 × 100 = 800,000 个训练 Token
```

这是便于理解的示例；真实训练还会受 Padding、被忽略位置和动态组批等因素影响。

## 常见误解

### 误解一：一个 Batch 一定对应一次参数更新

不一定。使用梯度累积时，多个 Micro-batch 才对应一次更新。

### 误解二：Batch Size 越大，模型一定越好

不一定。它会影响梯度估计、显存、吞吐和训练策略，但不是单独决定模型质量的参数。

### 误解三：一个 Epoch 对所有大模型都代表同样训练量

不是。它依赖数据集的定义与大小，因此不同项目之间不能只比较 Epoch 数。

### 误解四：Token Budget 越大，数据质量自然越高

不是。Token 数量只描述规模，内容质量、分布、重复度和配比同样重要。

## 理解检查

1. 为什么使用梯度累积时，一次 Forward 不一定等于一个 Step？
2. Epoch 和 Token Budget 分别适合描述什么？
3. 训练 Batch 与推理服务 Batch 的目标有什么不同？

## 继续学习

- 上一篇：[[00-训练进度与状态概览|训练进度与状态概览]]
- 下一篇：[[03-Checkpoint保存什么|Checkpoint 保存什么]]
