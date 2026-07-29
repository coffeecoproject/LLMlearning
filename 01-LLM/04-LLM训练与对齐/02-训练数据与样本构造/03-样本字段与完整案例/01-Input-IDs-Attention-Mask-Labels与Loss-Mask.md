---
type: concept
module: 1
status: complete
audience: non-specialist
reading: optional
parent: "[[00-训练样本字段与完整案例概览|训练样本字段与完整案例概览]]"
next: "[[03-从两篇文本到一个训练Batch|从两篇文本到一个训练 Batch]]"
tags: [llm, training-sample, input-ids, labels, mask]
---

# Input IDs、Attention Mask、Labels 与 Loss Mask

> [!summary]
> `input_ids` 决定模型读到哪些 Token，Attention 相关规则决定能读取哪些位置，`labels` 和 Loss Mask 决定哪些预测被拿来计算训练误差。

## 用一条样本先建立直觉

> [!example] 教学示意
> 假设正文被编码为 `[21, 35, 48, 2]`，其中 `2` 表示 EOS；所有数字均为虚构。

```text
input_ids:      [21, 35, 48,  2,  0]
attention_mask: [ 1,  1,  1,  1,  0]
labels:         [21, 35, 48,  2, -100]
```

第五个位置只是为了把长度补齐：

- `input_ids` 中放入 PAD 对应的 ID `0`；
- `attention_mask` 用 `0` 表示它不是有效正文；
- `labels` 用常见忽略值 `-100` 表示它不参与 Loss。

真实字段名称和忽略值取决于训练框架，这里展示的是常见形式。

## 1. input_ids：模型读什么

`input_ids` 是 Token ID 序列。模型用每个 ID 在 Embedding Matrix 中查找对应向量：

```text
input_ids
→ Embedding Lookup
→ 初始向量表示
```

若 Batch 中有 2 条、每条长度为 5，形状可以写为：

```text
[batch_size, sequence_length] = [2, 5]
```

## 2. attention_mask：哪些位置属于有效输入

在常见 Padding 示例中：

```text
1 → 有效 Token
0 → Padding 位置
```

它帮助模型避免把 PAD 当成正常内容读取。

但必须注意：用户在训练数据中看到的二维 `attention_mask`，通常主要标记有效 Token；Decoder-only 模型“不能看未来”的 Causal Mask 往往由模型内部另外生成或合并。

## 3. Causal Mask：当前位置能否看未来

```text
位置1只能看位置1
位置2可以看位置1和2
位置3可以看位置1、2和3
```

Causal Mask 解决的是时间方向限制；Padding Mask 解决的是占位位置无效。两者可能在内部合并成一个 Attention Bias，但概念职责不同。

| 规则 | 主要阻止什么 |
|---|---|
| Padding Mask | 读取补齐位置 |
| Causal Mask | 读取未来位置 |
| 文档隔离 Mask | 读取同一 Packed 序列中的其他文档 |

## 4. labels：拿什么作为正确目标

对于 Causal Language Modeling，目标是预测下一个 Token：

```text
看到 21      → 预测 35
看到 21, 35  → 预测 48
看到 21,35,48→ 预测 2
```

代码里常见两种表达：

### 表达方式 A：预先把目标错位

```text
输入： [21, 35, 48]
目标： [35, 48,  2]
```

### 表达方式 B：labels 与 input_ids 表面相同

```text
input_ids: [21, 35, 48, 2]
labels:    [21, 35, 48, 2]
```

然后模型内部在计算 Loss 时进行一位错位。Hugging Face 的 Causal LM 教程展示了这种常见做法。

来源：[Hugging Face Causal Language Modeling 官方教程](https://huggingface.co/docs/transformers/main/tasks/language_modeling)，核对日期：2026-07-28。

两种代码形式不同，但学习目标相同：当前位置预测后一个 Token。

## 5. Loss Mask：哪些位置真正计算误差

Loss Mask 是一个概念：选择哪些 Token 位置进入 Loss 汇总。

实现方式可能是：

- 单独提供 `loss_mask` 张量；
- 把不参与 Loss 的 `labels` 设为 `-100` 等忽略值；
- 使用训练框架的专用字段或权重。

例如一条 SFT 样本：

```text
User：2+2 等于几？
Assistant：4
```

若配方只训练 Assistant 回答：

```text
User 与格式 Token → Labels 设为忽略值
Assistant 的“4”   → 保留为训练目标
```

模型仍要读取 User 内容，才能生成回答；只是 User 位置不计入 Loss。

## 为什么“能读取”与“计算 Loss”必须分开

```text
能读取
→ 这个位置可作为上下文影响后续 Hidden State

计算 Loss
→ 这个位置对应的预测误差会影响参数更新
```

一段 Prompt 可以被模型读取，却不一定作为需要模仿生成的目标。这个区别对 SFT、工具轨迹和多轮对话训练非常重要。

## 常见误解

- **“attention_mask 就是 Causal Mask。”** 常见二维字段主要标记有效位置，未来限制通常由模型内部另外处理。
- **“labels 总要在数据文件中手工左移一位。”** 有些框架在模型内部完成错位。
- **“`-100` 是一个真实 Token ID。”** 它通常是 Loss 函数的忽略标记，不进入 Vocabulary。
- **“不计算 Prompt 的 Loss，就等于模型看不到 Prompt。”** Prompt 仍可作为上下文被读取。
- **“所有训练框架都有单独的 loss_mask 字段。”** 很多实现通过修改 Labels 达到相同效果。

## 理解检查

1. `input_ids` 与 `labels` 分别回答什么问题？
2. Padding Mask、Causal Mask 和 Loss Mask 的职责有什么不同？
3. 为什么一段 Prompt 可以参与 Attention，却不参与 Loss？
