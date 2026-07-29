---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Backward与参数更新概览|Backward 与参数更新概览]]"
previous: "[[../02-Forward与Loss/03-从Token-Loss到整体训练目标|从 Token Loss 到整体训练目标]]"
next: "[[02-Optimizer与Learning-Rate怎样更新参数|Optimizer 与 Learning Rate 怎样更新参数]]"
tags: [llm, training, backward, gradient, autograd]
---

# Backward 与 Gradient 分别是什么

> [!summary] 一句话理解
> `Backward`（反向传播）是从 Loss 出发，沿着刚才的计算路径反向追溯；`Gradient`（梯度）是追溯后得到的结果，用来表示每个参数怎样变化可能让 Loss 下降。

## 所属阶段

**训练阶段。** 普通模型运行只需要 Forward，不需要为模型参数执行 Backward。

## 为什么需要 Backward

Forward 已经完成了两件事：

1. 模型根据当前参数计算出预测；
2. Loss 衡量预测与目标之间的偏差。

但 Loss 只是一个结果，例如 `1.6`。它没有直接告诉系统：

- 是哪个参数对错误影响更大；
- 参数应该增大还是减小；
- 每个参数对 Loss 有多敏感。

Backward 的作用，就是把这个总误差逐层追溯回产生它的计算环节。

```text
参数
  ↓ Forward
Hidden States → Logits → Loss
                         ↓ Backward
Embedding参数 ← Attention参数 ← FFN参数 ← 输出层参数
```

图中箭头只是方向示意。真实模型包含大量层和大量参数，自动微分程序会沿实际计算图求出它们的梯度。

## Gradient 是什么

Gradient 可以先理解成：

> 如果这个参数发生一个很小的变化，Loss 大致会向哪个方向、变化多明显？

例如，某个参数当前是 `1.00`，计算得到梯度 `+0.40`。这表示在当前计算位置附近，增大这个参数倾向于让 Loss 增大；若希望降低 Loss，更新方向通常会与梯度相反。

如果梯度是负数，含义则相反。

> [!warning] 这是局部信息
> 梯度描述的是“当前参数、当前数据附近”的变化趋势，不是对所有数据都永远成立的规则。

## 一个极简示意

假设计算过程被简化为：

```text
参数 w → 预测结果 → Loss
```

Backward 得到：

```text
w 的梯度 = +0.40
```

这一步只产生信息：

- 当前参数：`1.00`
- 当前梯度：`+0.40`

参数此时仍然是 `1.00`，还没有被修改。真正决定更新幅度的是 [[02-Optimizer与Learning-Rate怎样更新参数|Optimizer 与 Learning Rate]]。

## Backward 实际怎样追溯

Forward 时，训练框架会记录与求梯度有关的计算关系，形成一张动态的“计算图”。Backward 从 Loss 开始，使用链式法则把影响逐层传回去。

不需要先掌握公式，但要抓住因果关系：

```text
某个参数影响了中间结果
→ 中间结果影响了 Logits
→ Logits 影响了 Loss
→ 因而可以追溯该参数对 Loss 的影响
```

如果一段操作没有参与当前 Loss 的计算，它通常不会从这个 Loss 得到有效梯度。

## 梯度会到达哪些参数

只要相关计算可求导并参与了当前预测，梯度就可能传到：

- 输出层参数；
- Transformer Block 中的 FFN、Attention 和归一化参数；
- Token Embedding 等更早的参数。

这并不表示每个参数收到相同梯度。不同参数参与预测的方式不同，得到的方向和大小也不同。

## Backward 不等于参数更新

这是最重要的边界：

| 环节 | 做什么 | 是否修改参数 |
|---|---|---:|
| Forward | 用当前参数计算预测 | 否 |
| Loss | 衡量预测偏差 | 否 |
| Backward | 计算各参数的梯度 | 否 |
| Optimizer Step | 根据梯度更新参数 | 是 |

把 Backward 和更新分开后，系统才可以在更新前进行梯度累积、缩放或裁剪。

## 常见误解

### 误解一：Gradient 就是参数改变量

不是。Gradient 是更新依据；实际改多少还取决于学习率、优化器状态、权重衰减等设置。

### 误解二：Backward 会重新倒着生成文本

不是。“反向”指误差信号沿计算关系反向传播，与文本生成顺序无关。

### 误解三：运行模型也会自动反向传播

普通运行通常只读取参数生成结果，不计算模型参数梯度，也不修改参数。

### 误解四：一次梯度就说明模型应该学到什么知识

单个 Batch 的梯度只反映这批训练样本产生的局部信号。稳定能力来自大量样本和连续更新的共同作用。

## 选读：梯度过小或过大

- 梯度过小，前面层的参数可能很难得到有效更新，称为梯度消失；
- 梯度过大，更新可能不稳定，称为梯度爆炸。

Transformer 中的 Residual、Normalization、初始化和梯度裁剪等设计，都与训练稳定性有关。这里先建立边界，不展开工程细节。

## 开放实现观察

PyTorch Autograd 会记录参与求梯度的计算关系，并由 `backward()` 计算输出相对于相关参数的 Gradient。部分 Forward 中间结果需要保留到 Backward 使用，这也是训练 Forward 通常比普通运行占用更多内存的原因之一。

来源：[PyTorch Autograd](https://docs.pytorch.org/docs/stable/autograd.html)，核对日期：2026-07-28。

## 理解检查

1. Loss 已经给出误差，为什么还需要 Backward？
2. Backward 完成后，参数是否已经变化？
3. 为什么说梯度只是当前位置附近的局部信息？

## 继续学习

- 上一篇：[[00-Backward与参数更新概览|Backward 与参数更新概览]]
- 下一篇：[[02-Optimizer与Learning-Rate怎样更新参数|Optimizer 与 Learning Rate 怎样更新参数]]
