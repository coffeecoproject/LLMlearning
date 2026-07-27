---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer基础机制概览|Output-Layer基础机制概览]]"
previous: "[[03-Logits是什么|Logits是什么]]"
next: "[[05-训练与运行怎样使用Logits|训练与运行怎样使用Logits]]"
tags: [llm, softmax, probability, logits]
---

# Softmax 怎样把分数变成概率

> [!summary]
> Softmax 把一组任意大小的 Logits 转换成一组非负且总和为 1 的数，同时保留“原分数越高，所得概率通常越高”的顺序关系。

## 它解决什么问题

LM Head 给出的原始分数可能是：

```text
[2.0, 1.0, 0.0]
```

这些数不能直接叫概率。Softmax 把它们转换成大致：

```text
[0.665, 0.245, 0.090]
```

现在三项：

- 都不小于 0；
- 总和约等于 1；
- 原本分数最高的第一项仍有最高概率。

完整小数字过程见 [[03-Logits到概率的小数字示例|Logits 到概率的小数字示例]]。

## 为什么不是简单除以总和

Logits 可能包含负数，而且总和可能是 0。直接相除既不能保证非负，也可能无法计算。Softmax 会先通过指数转换把每一项变成正数，再按总和归一化。

主线阅读不要求记公式，只需理解：

```text
放大相对差距并变成正数
→ 再让全部候选总和变成 1
```

## Softmax 是不是可学习层

标准 Softmax 本身没有训练形成的权重。它是一种固定的数值转换。真正学习到“什么上下文下哪个候选分数更高”的主要是前面的模型参数，包括 Transformer 与 LM Head 权重。

## 训练阶段与运行阶段的区别

### LLM 训练阶段

常用 Cross Entropy Loss 通常直接接收 Logits，并在内部使用与 Log-Softmax 等价的稳定计算。代码里没有显式写出一层 Softmax，不表示概率关系消失了。

### LLM 运行阶段

生成系统可能先对 Logits 进行 Temperature、屏蔽不允许的 Token 或其他处理，再计算概率并采样。若采用贪心选择，也可以直接比较处理后的 Logits 大小，不必为了找最大值而完整计算 Softmax。

## 概率不等于事实置信度

Softmax 概率表示“在当前上下文、模型参数和候选词表下，下一个 Token 的相对分布”。它不直接表示：

- 这句话在现实中有多大概率为真；
- 模型是否拥有可靠证据；
- 整段回答最终有多高正确率。

## 理解检查

1. Softmax 为什么不能用简单除以 Logits 总和替代？
2. Softmax 本身有没有一套通过训练形成的大矩阵？
3. 下一个 Token 概率为 0.7，是否等于这句话有 70% 概率为真？

下一篇：[[05-训练与运行怎样使用Logits|训练与运行怎样使用 Logits]]。
