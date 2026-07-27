---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-输入准备概览|输入准备概览]]"
previous: "[[01-Prompt-Messages与Chat-Template|Prompt-Messages与Chat-Template]]"
next: "[[00-Prefill与首次预测概览|Prefill与首次预测概览]]"
tags: [llm, inference, tokenizer, input-ids, attention-mask]
---

# Tokenizer 与模型输入张量

> [!summary]
> Chat Template 准备好序列后，配套 Tokenizer 把它变成 `input_ids`；运行软件还会准备 Mask、长度或位置信息，让模型知道哪些位置有效以及各位置的顺序。

## 最核心的输入是什么

对文本 Decoder-only LLM，最核心的是：

```text
input_ids = [151643, 9707, 198,  ...]
```

每个整数都是目标模型 Vocabulary 中一个 Token 的编号。Embedding Lookup 根据这些 ID 取得初始向量，之后才进入 Transformer。

> [!warning] 数字边界
> 上面仅展示数据形态，不代表任何一句具体文本在某个模型中的真实编码。

## 为什么还有其他输入信息

常见运行输入包括：

| 输入 | 作用 | 是否总由用户显式提供 |
|---|---|---|
| `input_ids` | 指出序列由哪些 Token 组成 | 通常会提供 |
| `attention_mask` | 区分有效 Token 与补齐位置 | 可能由 Tokenizer/Runtime 自动生成 |
| `position_ids` 或等价状态 | 指出 Token 位置顺序 | 经常由模型或 Runtime 自动推导 |

这里要区分两种 Mask：

- **Padding Mask**：批处理时忽略为对齐长度而补出的空位置；
- **Causal Mask**：禁止当前位置读取未来 Token，通常由模型 Attention 实现。

二者都叫 Mask，但解决的问题不同。

## 单请求也可能出现 Batch 维度

即使一次只处理一条输入，软件也常保持统一形状：

```text
[batch_size, sequence_length]
= [1, 8]
```

这里 `batch_size = 1` 只表示当前张量里有一条序列，不表示系统没有 Batch 机制。

如果一批序列长度不同，运行软件可能使用 Padding 和 Mask 对齐，也可能通过更紧凑的内部表示减少浪费。这属于实现选择；逻辑上每条请求仍有自己的有效长度。

## 输入怎样接回基础结构

```text
input_ids
→ Embedding Lookup
→ 初始表示与 Position 信息
→ Transformer Blocks
→ Output Layer
→ Logits
```

因此，前一部分学过的 Tokenizer、Embedding、Position、Transformer 和 Output Layer 并没有被替换；本部分研究的是它们怎样被组织成一次持续生成。

## 常见误解

- `input_ids` 是 Token 的编号，不是 Embedding 向量本身。
- `attention_mask` 不等于 Attention Weight。
- Batch 维度存在，不代表单个回答中的多个未来 Token 能同时确定。
- 模型输入张量是运行软件和模型的接口，不应全部理解成用户可调的模型参数。

## 理解检查

1. `input_ids` 怎样连接 Tokenizer 和 Embedding？
2. Padding Mask 和 Causal Mask 分别防止什么？
3. 为什么单请求的输入形状也常有 `batch_size = 1`？

下一部分：[[00-Prefill与首次预测概览|Prefill 与首次预测]]。
