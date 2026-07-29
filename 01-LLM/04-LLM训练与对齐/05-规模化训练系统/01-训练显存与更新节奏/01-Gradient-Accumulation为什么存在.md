---
type: concept
module: 1
status: complete
audience: non-specialist
reading: optional
parent: "[[00-训练显存与更新节奏概览|训练显存与更新节奏概览]]"
tags: [llm, training, gradient-accumulation, micro-batch, effective-batch]
---

# Gradient Accumulation 为什么存在

> [!summary] 一句话理解
> Gradient Accumulation（梯度累积）让系统分几次处理较小的 Micro-batch，先累积 Gradient，再统一更新参数，以缓解单次显存压力。

## 所属阶段

**训练工程。** 它不是模型层，也不同于推理服务把多个用户请求放入同一个 Batch。

## 为什么需要它

训练 Forward 和 Backward 要保留许多中间结果。目标 Batch 太大时，一次可能放不进 GPU 显存。

```text
Micro-batch 1 → Forward → Backward → 保留 Gradient
Micro-batch 2 → Forward → Backward → 累加 Gradient
Micro-batch 3 → Forward → Backward → 累加 Gradient
                                      ↓
                               Optimizer Step
```

参数只在 Optimizer Step 时更新。

## 一个简单例子

```text
每个 Micro-batch：2 个样本
累积次数：4
一次更新综合：2 × 4 = 8 个样本
```

这 8 个样本构成的规模可理解为 Effective Batch。多设备训练还需要考虑设备数量。

## 它解决什么，不解决什么

它主要缓解单次内存压力，但不会凭空减少总计算量。一次参数更新仍需要完成多次 Forward 和 Backward。

正确实现还要保证 Loss 或 Gradient 的缩放符合目标 Batch 配方。这里不进入代码细节。

## 与推理 Batch 的区别

| 梯度累积 | 推理服务 Batch |
|---|---|
| 训练阶段 | 运行阶段 |
| 累积 Gradient | 共同执行模型计算 |
| 最后更新参数 | 不更新参数 |
| 主要服务于显存和训练配置 | 主要服务于吞吐和调度 |

## 常见误解

- 多次 Backward 不代表参数每次都变化；
- 累积的是 Gradient，不是模型生成的文本；
- 累积次数越多不一定越好；
- 它用于接近目标 Effective Batch，但真实结果还会受随机性和数值实现影响。

## 理解检查

1. 为什么多次 Backward 可以只对应一次参数更新？
2. 梯度累积主要缓解内存问题还是总计算量问题？
3. 它为什么不等于推理服务的 Batch？
