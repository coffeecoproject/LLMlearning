---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-预测目标与Labels概览|预测目标与 Labels 概览]]"
previous: "[[01-Causal-Language-Modeling与下一Token目标|Causal Language Modeling 与下一 Token 目标]]"
next: "[[00-Forward与Loss概览|Forward 与 Loss 概览]]"
tags: [llm, labels, shift, teacher-forcing]
---

# Labels 错位与 Teacher Forcing

> [!summary]
> Labels 与输入在概念上错开一个位置，使当前位置的预测对照真实下一个 Token；Teacher Forcing 则表示训练时每个位置使用真实前缀，而不是模型前一步可能生成错的 Token。

## 为什么需要错位一位

> [!example] 教学示意

```text
完整序列：我 | 喜欢 | 苹果 | EOS
```

如果模型位置和 Label 完全按同一含义比较：

```text
看到“我” → 目标仍是“我”
```

模型只需要复述当前 Token，而不是预测后续。

真正需要的是：

```text
输入位置：我     | 喜欢 | 苹果
目标位置：喜欢   | 苹果 | EOS
```

这就是 Shifted Labels，中文可理解为“向后错位一位的目标”。

## 为什么代码里 Labels 有时看起来没有错位

常见实现有两种：

### 方式一：数据管线提前构造

```text
input_ids = [我, 喜欢, 苹果]
labels    = [喜欢, 苹果, EOS]
```

### 方式二：模型计算 Loss 时内部错位

```text
input_ids = [我, 喜欢, 苹果, EOS]
labels    = [我, 喜欢, 苹果, EOS]
```

模型内部使用：

```text
前面的 Logits
↔ 后一个位置的 Labels
```

所以不能只看两个数组表面是否相同，就判断模型在“预测自己”。要查看 Loss 计算发生在哪一步。

Hugging Face 的 Causal LM 官方教程展示了第二种常见形式。

来源：[Hugging Face Causal Language Modeling](https://huggingface.co/docs/transformers/main/tasks/language_modeling)，核对日期：2026-07-28。

## Teacher Forcing 是什么

Teacher Forcing 不是额外的教师大模型。这里的“Teacher”是训练数据中已经存在的真实 Token。

训练序列：

```text
我 | 喜欢 | 苹果
```

即使模型在第一个位置更想预测“讨厌”，计算下一个位置时仍使用真实前缀：

```text
我 喜欢
```

而不是把模型错误预测的：

```text
我 讨厌
```

继续喂回去。

## 为什么训练要使用真实前缀

如果训练早期模型几乎随机，第一步预测错后继续使用错误输出：

```text
第一步错
→ 后续上下文越来越偏
→ 很难稳定学习每个正确位置的关系
```

使用真实前缀可以让每个位置都得到相对明确的训练任务，并支持多个位置并行计算。

## 与普通运行有什么区别

### 训练阶段

```text
已知完整真实序列
→ 每个位置使用真实前缀
→ 计算多个位置的 Loss
```

### 普通运行阶段

```text
只有 Prompt
→ 模型选出下一个 Token
→ 把自己刚生成的 Token 加入上下文
→ 继续生成
```

因此运行时一次错误输出可能影响后续生成路径，而标准 Teacher Forcing 训练中不会用这次错误继续构造后续前缀。

## Exposure Bias 是什么

这是一个选读边界：训练主要面对真实前缀，运行主要面对模型自己生成的前缀，两者分布并不完全相同，这种差异常被称为 Exposure Bias。

当前只需知道它解释了：

> 模型在单个局部预测上训练得不错，也可能在长时间自回归生成中逐渐偏离。

后训练、数据设计和生成策略会尝试缓解相关问题，但没有一个单独机制能完全消除。

## Labels 中为什么有忽略位置

Padding、某些 Special Token 或只读取不训练的 Prompt 位置，可能被设为 `-100` 等忽略值：

```text
有效目标 → 计算 Loss
忽略目标 → 不进入 Loss 汇总
```

这就是前一专题讲过的 Loss Mask 常见实现。

## 常见误解

- **“Teacher Forcing 是用另一个教师模型指导。”** 这里使用的是训练数据中的真实历史 Token。
- **“labels 和 input_ids 相同就没有错位。”** 错位可能在模型内部完成。
- **“模型预测错一次，训练后续位置就都使用错误内容。”** 标准 Teacher Forcing 继续使用真实前缀。
- **“运行时也能获得真实未来 Token。”** 运行时未来输出尚不存在，只能使用模型已生成内容。

## 理解检查

1. 如果 Labels 不错位，模型可能学成什么任务？
2. 为什么 `labels = input_ids.copy()` 仍然可能训练下一 Token 预测？
3. Teacher Forcing 为什么帮助训练，也为什么与运行状态存在差异？
