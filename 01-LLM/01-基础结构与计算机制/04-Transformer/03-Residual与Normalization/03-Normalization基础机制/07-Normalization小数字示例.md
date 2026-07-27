---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Normalization基础机制概览|Normalization基础机制概览]]"
previous: "[[06-Block内Norm与Final-Norm|Block内Norm与Final-Norm]]"
next: "[[00-FFN概览|FFN概览]]"
tags: [llm, normalization, worked-example, optional]
---

# Normalization 小数字示例

> [!summary]
> 对同一个小向量分别执行简化 LayerNorm 与 RMSNorm，可以直观看到：一个会先减均值，一个直接按整体平方幅度缩放。

> [!warning] 教学示例
> 下面使用人为设计的小向量，并暂时忽略 `eps`；设可学习 Scale 全为 1、Bias 全为 0。数字不来自真实模型。

## 输入向量

假设一个 Token 位置的 Hidden State 是：

```text
x = [1, 2, 3]
```

这只是一个位置的 3 维向量，不是三个 Token。

## 简化 LayerNorm

### 1. 求均值

```text
均值 = (1 + 2 + 3) ÷ 3 = 2
```

### 2. 每一维减去均值

```text
[1,2,3] - 2
→ [-1,0,1]
```

### 3. 根据分散程度缩放

平方偏差平均：

```text
(1² + 0² + 1²) ÷ 3
= 2/3
```

其平方根约为 `0.816`。因此：

```text
[-1,0,1] ÷ 0.816
≈ [-1.225, 0, 1.225]
```

重点不是记小数，而是看到 LayerNorm 先把向量围绕均值 0 居中。

## 简化 RMSNorm

### 1. 各维度平方并求平均

```text
(1² + 2² + 3²) ÷ 3
= (1 + 4 + 9) ÷ 3
= 14/3
```

### 2. 开平方得到 RMS

```text
sqrt(14/3) ≈ 2.160
```

### 3. 原向量直接除以 RMS

```text
[1,2,3] ÷ 2.160
≈ [0.463, 0.926, 1.389]
```

RMSNorm 没有先减去均值，所以三个结果仍然都为正。

## 并排观察

```text
输入：               [1, 2, 3]
简化 LayerNorm：≈ [-1.225, 0, 1.225]
简化 RMSNorm：  ≈ [0.463, 0.926, 1.389]
```

两者都控制尺度，但处理方式不同。

## 真实模型还多了什么

- `eps`：防止分母过小；
- 可学习 Scale；
- LayerNorm 可能还有可学习 Bias；
- 高维向量与低精度数值实现；
- 针对硬件优化的融合算子。

这些不会改变当前要理解的核心差别。

## 理解检查

1. 为什么 LayerNorm 示例中出现了负数，而 RMSNorm 示例仍全为正？
2. `x=[1,2,3]` 表示三个 Token 还是一个 Token 的三个维度？
3. 可学习 Scale 在训练和运行阶段分别怎样处理？

Normalization 基础机制完成。下一节：[[00-FFN概览|FFN / MLP]]。
