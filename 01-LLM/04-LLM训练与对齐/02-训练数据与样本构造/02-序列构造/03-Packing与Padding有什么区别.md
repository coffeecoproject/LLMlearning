---
type: concept
module: 1
status: complete
audience: non-specialist
reading: optional
parent: "[[00-序列构造概览|序列构造概览]]"
previous: "[[02-切段与文档边界怎样处理|切段与文档边界怎样处理]]"
next: "[[00-训练样本字段与完整案例概览|训练样本字段与完整案例概览]]"
tags: [llm, training, packing, padding]
---

# Packing 与 Padding 有什么区别

> [!summary]
> Packing 用其他真实 Token 填满空余训练位置；Padding 用无训练内容的占位 Token 补齐形状。前者提高有效利用率，后者方便组成规则张量。

## 先看同一个问题

假设训练序列长度设为 8，目前有三段文本：

```text
文档A：3 Token
文档B：2 Token
文档C：7 Token
```

### 只使用 Padding

```text
A：[A A A PAD PAD PAD PAD PAD]
B：[B B PAD PAD PAD PAD PAD PAD]
C：[C C C C C C C PAD]
```

三条序列形状一致，但许多位置没有真实训练内容。

### 使用 Packing

```text
序列1：[A A A EOS B B EOS PAD]
序列2：[C C C C C C C EOS]
```

短文档被组合进同一训练序列，真实 Token 比例明显提高。

> [!example] 教学示意
> 示例忽略了具体 Tokenizer 切分和模型的边界规则，只用于比较两种操作。

## Padding 解决什么问题

批量矩阵计算通常希望一个 Batch 里的序列形状一致：

```text
[batch_size, sequence_length]
```

如果两条序列长度分别是 5 和 8，可以把短序列补到 8。补出的 PAD 位置通常：

- 不应当作为真实输入参与 Attention；
- 不应当作为预测目标计算 Loss；
- 需要由 Attention Mask 或 Labels 忽略规则标记。

Padding 的主要价值是形状对齐，不是给模型增加知识。

## Packing 解决什么问题

Packing 将多个短样本放进一个长度上限内：

```text
多个短样本
→ 用边界标记分隔
→ 尽可能填满固定长度
```

它减少无效 PAD 计算，尤其适合大量短文本或短对话样本。

但 Packing 需要额外决定：

- 文档之间插入什么边界 Token；
- 是否允许跨样本 Attention；
- Position 是否连续还是重置；
- 跨边界位置是否计入 Loss；
- 一条样本是否允许被拆开。

## 两者可以同时存在吗

可以。

Packing 后最后仍可能剩少量空位：

```text
[文档A | EOS | 文档B | EOS | PAD]
```

因此真实样本可能先 Packing，再对仍未填满的尾部 Padding。

## 动态 Padding 与固定 Padding

### 固定到全局最大长度

所有 Batch 都补到预设最大长度，逻辑简单，但短 Batch 可能浪费很多计算。

### 动态补到当前 Batch 最长序列

```text
本 Batch 最长长度 = 120
→ 其他序列只补到 120
```

通常比统一补到更大上限节省计算。Hugging Face 官方 Data Collator 文档就提供了补到当前 Batch 最长序列的策略。

来源：[Hugging Face Data Collator 官方文档](https://huggingface.co/docs/transformers/main_classes/data_collator)，核对日期：2026-07-28。

## Packing 不会改变模型上下文上限

如果训练序列上限是 8192 Token：

```text
Packing 前：多个短样本
Packing 后：一个不超过 8192 Token 的组合序列
```

Packing 只是利用已有窗口，不会把 8192 自动扩大为更长上下文。

## 常见误解

- **“Packing 就是 Padding 的另一种名称。”** Packing 填入真实样本，Padding 填入占位位置。
- **“用了 Packing 就不会再出现 PAD。”** 组合后仍可能留有无法利用的尾部空间。
- **“PAD Token 会作为正常知识被模型学习。”** 正确构造通常会屏蔽它的 Attention 或 Loss；配置错误才可能产生污染。
- **“把多个样本放一起就一定互相可见。”** 是否跨样本 Attention 由具体 Mask 和位置规则决定。

## 理解检查

1. Packing 和 Padding 分别想减少或解决什么？
2. 为什么 Padding 后必须配套 Mask 或忽略规则？
3. Packing 为什么不会自动扩大模型的上下文窗口？
