---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Residual与Normalization概览]]"
previous: "[[Residual与Normalization概览]]"
next: "[[Residual基础机制概览]]"
tags: [llm, residual, normalization, framework, beginner]
---

# Residual 与 Normalization 一页看懂

> [!summary]
> Residual 保留 Hidden State 主干并把子层变化加回来；Normalization 调整数值尺度。它们共同组织深层 Block 的稳定数据流，但职责不同。

## 它们位于哪里

常见 Pre-Norm Block 可以先简化为：

```text
输入
→ Norm → Attention → Residual 相加
→ Norm → FFN       → Residual 相加
→ 输出
```

真实模型也可能采用 Post-Norm 或并行结构，但 Residual 和 Normalization 都位于 Transformer Block 内部或紧邻其数据通路。

## Residual 做什么

```text
旧 Hidden State
+ 子层产生的变化
→ 新 Hidden State
```

它让 Attention 或 FFN 不必独自重建完整表示，而可以在保留主干的基础上提交一次变化。

## Normalization 做什么

多层计算会不断改变向量数值。Normalization 对一组数值进行尺度整理，使后续子层接收到更可控的输入。

它不会把文字“规范化”，也不是 Tokenizer 中的文本 Normalization。

## 两者为什么不能合并

```text
Residual
→ 解决旧表示与新变化怎样连接

Normalization
→ 解决进入子层的数值尺度怎样整理
```

一个是信息通路和相加关系，一个是数值变换。

## 阶段边界

> [!info] 两阶段共同
> LLM 训练和运行都会执行 Residual 与 Normalization。训练还会更新 Norm 的可学习参数并利用 Residual 改善梯度传播；普通运行只执行固定前向计算。

## 框架层检查

1. Residual 为什么不是与 Attention 并列的第三种信息检索模块？
2. Normalization 管理的是文字格式还是向量数值尺度？
3. 一个常见串行 Block 为什么通常有两次 Residual 相加？

能回答这三题，就可以进入 [[FFN框架速览概览|FFN 一页看懂]]。需要理解 Residual 细节时，再读 [[Residual基础机制概览|Residual 基础机制]]。
