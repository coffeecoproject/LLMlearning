---
type: worked-example
module: 1
status: complete
audience: non-specialist
reading: optional
parent: "[[00-训练样本字段与完整案例概览|训练样本字段与完整案例概览]]"
previous: "[[01-Input-IDs-Attention-Mask-Labels与Loss-Mask|训练样本中的关键字段]]"
next: "[[00-训练目标与参数更新概览|训练目标与参数更新概览]]"
tags: [llm, training-data, batch, worked-example]
---

# 从两篇文本到一个训练 Batch

> [!summary]
> 一条训练数据管线会先决定资料能否使用，再把正文清理、去重、混合、编码、切段和装箱，最终产出形状一致且目标位置明确的 Batch。

## 示例设定

> [!example] 全部是教学示意
> 为了看清结构，假设训练序列长度为 8，并虚构 Token 与 Token ID；这不是任何真实模型的切分结果。

候选数据：

```text
文档A：苹果是水果。
文档B：香蕉是水果。
文档C：某网站转载的“苹果是水果。”
```

## 第一步：来源与许可记录

训练系统给每篇文档保存：

```text
来源、获取时间、版本、内容哈希、许可或使用依据、处理记录
```

如果文档C来源不清或不满足项目规则，它可能在这里被排除；即使允许继续，也要进入后面的去重。

## 第二步：解析与质量过滤

假设原始网页实际是：

```text
首页 | 下载App | 苹果是水果。 | 广告 | 联系我们
```

解析后保留：

```text
苹果是水果。
```

同时检查语言、乱码、隐私、凭据和项目安全规则。

## 第三步：去重

```text
文档A：苹果是水果。
文档C：苹果是水果。
```

两者正文相同，因此只保留符合来源和质量规则的代表版本。

最终保留：

```text
文档A：苹果是水果。
文档B：香蕉是水果。
```

## 第四步：进入数据混合

假设本训练批次的数据配方决定抽到两条中文通用文本。这不是按文件自然顺序随意读取，而是数据混合与采样结果的一部分。

## 第五步：固定 Tokenizer 编码

假设得到：

```text
文档A：
[苹果] [是] [水果] [。] [EOS]
[  11] [20] [  35] [ 7] [  2]

文档B：
[香蕉] [是] [水果] [。] [EOS]
[  18] [20] [  35] [ 7] [  2]
```

每篇文档各有 5 个 Token。

## 第六步：决定 Packing 和 Padding

序列长度只有 8，两篇文档合计 10 Token，不能完整放入同一条序列。

一种简单策略是暂不跨文档拆分：

```text
样本1：[11, 20, 35, 7, 2, PAD, PAD, PAD]
样本2：[18, 20, 35, 7, 2, PAD, PAD, PAD]
```

另一种更复杂的 Packing 管线可能把不同短文与其他文本组合，但必须同时处理文档边界和 Attention 规则。

本例选择第一种，以便观察 Padding。

## 第七步：构造字段

假设 PAD ID 为 `0`，忽略 Loss 的值为 `-100`：

```text
input_ids =
[
  [11, 20, 35, 7, 2, 0, 0, 0],
  [18, 20, 35, 7, 2, 0, 0, 0]
]

attention_mask =
[
  [ 1,  1,  1, 1, 1, 0, 0, 0],
  [ 1,  1,  1, 1, 1, 0, 0, 0]
]

labels =
[
  [11, 20, 35, 7, 2, -100, -100, -100],
  [18, 20, 35, 7, 2, -100, -100, -100]
]
```

形状是：

```text
[batch_size, sequence_length] = [2, 8]
```

## 第八步：理解真正的预测目标

虽然 `labels` 表面与有效 `input_ids` 相同，Causal LM 在计算 Loss 时会进行一位错位：

```text
看到“苹果”       → 预测“是”
看到“苹果 是”    → 预测“水果”
看到“苹果 是 水果”→ 预测“。”
看到完整句子      → 预测 EOS
```

PAD 位置不参与 Loss。Causal Mask 还会确保前面位置无法读取未来 Token。

## 第九步：本专题在这里停止

到这里，训练管线已经准备好 Batch：

```text
Batch
→ input_ids、attention_mask、labels
```

下一专题才会继续：

```text
Forward
→ Logits
→ Loss
→ Backward
→ Gradient
→ Optimizer 更新参数
```

进入：[[00-训练目标与参数更新概览|训练目标与参数更新]]。

## 如果这是 SFT 样本会怎样变化

假设样本是：

```text
User：苹果是什么？
Assistant：水果。
```

还需要先应用模型的 Chat Template；若训练配方只学习 Assistant 回答，则 User、格式和 Padding 位置的 `labels` 都会被设为忽略值，只有 Assistant 内容参与 Loss。

## 最终复习

```text
原始资料
→ 来源与许可
→ 解析和过滤
→ 去重
→ 数据混合
→ 固定 Tokenizer
→ 文档边界与切段
→ Packing / Padding
→ input_ids / attention_mask / labels
→ Batch
→ 等待模型训练计算
```

## 理解检查

1. 为什么文档C即使内容正确，也不一定应作为独立样本保留？
2. `attention_mask` 中的 0 与 `labels` 中的 `-100` 分别控制什么？
3. 为什么 `labels` 表面与 `input_ids` 相同时，仍然可以学习下一个 Token？
4. 这个 Batch 形成后，模型参数是否已经发生变化？
