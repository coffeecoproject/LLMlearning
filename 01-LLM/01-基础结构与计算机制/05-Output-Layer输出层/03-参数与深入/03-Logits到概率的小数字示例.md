---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer参数与深入概览|Output-Layer参数与深入概览]]"
previous: "[[02-Weight-Tying是什么|Weight-Tying是什么]]"
next: "[[04-真实模型Output-Layer观察|真实模型Output-Layer观察]]"
tags: [llm, lm-head, logits, softmax, math]
---

# Logits 到概率的小数字示例

> [!summary]
> 一个位置的 Hidden State 先通过 LM Head 得到每个词表候选的 Logit，再由 Softmax 把这些相对分数转换成总和为 1 的概率。

> [!warning] 教学示意
> 以下 Token、向量和权重全部为虚构小数字，只用于展示计算关系，不来自任何真实模型。

## 第一步：准备一个极小词表

```text
Token A：猫
Token B：狗
Token C：<EOS>

vocab_size = 3
hidden_size = 2
```

当前最后一个位置的 Hidden State 是：

```text
h = [2, 1]
```

## 第二步：LM Head 为三个候选打分

假设输出权重中，三个候选方向分别是：

```text
猫：   [ 1, 0]
狗：   [ 0, 1]
<EOS>：[-1, 1]
```

分别做最简单的加权组合：

```text
猫：   2×1 + 1×0  =  2
狗：   2×0 + 1×1  =  1
<EOS>：2×(-1)+1×1 = -1
```

所以 Logits 是：

```text
[2, 1, -1]
```

这一步输出的是原始分数，不是概率。

## 第三步：Softmax 转成概率

可选数学轮廓：先对每个 Logit 取指数，再除以指数之和。

```text
e²  ≈ 7.39
e¹  ≈ 2.72
e⁻¹ ≈ 0.37

总和 ≈ 10.48
```

于是：

```text
猫：    7.39 / 10.48 ≈ 0.705
狗：    2.72 / 10.48 ≈ 0.259
<EOS>： 0.37 / 10.48 ≈ 0.035
```

由于四舍五入，总和可能显示为 0.999 或 1.001；理论上总和为 1。

## 第四步：怎样使用

### 训练阶段

如果正确的下一个 Token 是“狗”，Loss 会发现模型把更高概率给了“猫”，从而形成误差信号。参数怎样更新属于训练专题。

### 运行阶段

生成策略可以：

- 贪心选择最高项“猫”；
- 根据概率随机采样，仍有可能选到“狗”或 `<EOS>`；
- 先用 Temperature、Top-k 等修改分布后再选择。

因此“概率最高”不等于“任何策略下一定被选中”。

## 从例子中必须看懂什么

```text
Hidden State：[hidden_size]
→ LM Head
→ Logits：[vocab_size]
→ Softmax
→ Probabilities：[vocab_size]
```

矩阵计算的目的不是增加神秘步骤，而是让一个上下文向量能够分别对整套词表候选打分。

## 理解检查

1. 为什么 `[-1]` 这个 Logit 最终仍得到正概率？
2. LM Head 计算前后，最后一维从什么变成了什么？
3. 为什么概率最高的 Token 在采样策略下仍可能不被选择？

下一篇：[[04-真实模型Output-Layer观察|真实模型 Output Layer 观察]]。
