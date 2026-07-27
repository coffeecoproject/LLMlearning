---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[MHA-GQA与MQA概览]]"
previous: "[[MHA-GQA与MQA完整对比]]"
next: "[[MLA与注意力变体概览]]"
tags: [llm, model-config, qwen3, gpt-oss, gqa]
---

# 怎样从模型配置判断 Head 结构

> [!summary]
> 对使用常见配置约定的模型，可以比较 `num_attention_heads` 和 `num_key_value_heads`：二者相等通常是标准 MHA，KV Head 为 1 通常是 MQA，KV Head 位于两者之间通常是 GQA。

## 先认识三个字段

```text
num_attention_heads
→ 通常表示 Query Head 数量

num_key_value_heads
→ 通常表示 Key Head 和 Value Head 数量

head_dim
→ 每个 Head 的向量宽度
```

这些字段名是常见约定，不是跨所有代码库的强制标准。判断真实模型时还要结合对应版本的模型类和实现。

## 常见判断规则

令：

```text
Hq = num_attention_heads
Hkv = num_key_value_heads
```

常见情况是：

```text
Hkv = Hq
→ 标准 MHA

1 < Hkv < Hq
→ GQA

Hkv = 1
→ MQA
```

如果配置没有 `num_key_value_heads`，有些实现会默认 K/V Head 数量等于 Query Head 数量，也有实现使用其他字段；不能在没有模型实现证据时一概而论。

## 开放模型观察一：Qwen3-8B

Qwen3-8B 官方 `config.json` 公布：

```json
{
  "hidden_size": 4096,
  "head_dim": 128,
  "num_attention_heads": 32,
  "num_key_value_heads": 8
}
```

### 直接公开的事实

```text
Query Head = 32
KV Head = 8
head_dim = 128
```

### 根据字段做出的结构推导

```text
32 ÷ 8 = 4
→ 属于 GQA
→ 每个 KV Head 对应 4 个 Query Head
```

此外：

```text
32 × 128 = 4096 = hidden_size
```

在这个版本中，Query Head 拼接宽度与主 Hidden State 宽度相同。

## 开放模型观察二：OpenAI gpt-oss-20b

OpenAI `gpt-oss-20b` 官方 `config.json` 公布：

```json
{
  "hidden_size": 2880,
  "head_dim": 64,
  "num_attention_heads": 64,
  "num_key_value_heads": 8
}
```

### 直接公开的事实

```text
Query Head = 64
KV Head = 8
head_dim = 64
hidden_size = 2880
```

### 根据字段做出的结构推导

```text
64 ÷ 8 = 8
→ 属于 GQA
→ 每个 KV Head 对应 8 个 Query Head
```

还有一个重要边界：

```text
64 × 64 = 4096
4096 ≠ hidden_size 2880
```

这说明 `num_attention_heads × head_dim = hidden_size` 是常见教学关系，却不是所有真实模型都必须满足。根据公开字段可以推断该模型的 Query Head 总宽度为 4096；具体投影布局仍应结合官方实现，而不能只用 Qwen 的形状经验套用。

## 为什么这两个例子有价值

```text
Qwen3-8B
→ 展示常见的 32 Query / 8 KV GQA
→ Query Head 总宽度等于 hidden_size

OpenAI gpt-oss-20b
→ 展示 64 Query / 8 KV GQA
→ Query Head 总宽度不等于 hidden_size
```

因此，读取配置应分两步：

```text
先判断 Query Head 与 KV Head 的数量关系
再分别核对 head_dim、hidden_size 和具体实现
```

## 对闭源模型能否这样判断

只有在对应模型版本公开这些架构字段时，才能据此判断。对于没有公开内部配置的闭源 API 模型，不能因为它由某家公司提供，就把同公司的开放模型配置套上去。

OpenAI `gpt-oss` 可以作为 OpenAI 开放权重模型的可验证观察，但不能据此断言其他未公开 GPT 模型使用完全相同的 Head 结构。

## 配置不能告诉你的内容

仅凭这三个字段，不能完整判断：

- 各 Head 实际学到了什么；
- 模型回答质量；
- KV Cache 的具体内存布局；
- Attention 内核如何广播或复制 K/V；
- 实际 Prefill 和 Decode 速度；
- 是否还有局部窗口、Attention Sink 或其他变体。

配置提供的是结构证据，不是完整运行报告。

## 官方来源

- [Qwen3-8B 官方 `config.json`](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)
- [OpenAI `gpt-oss-20b` 官方 `config.json`](https://huggingface.co/openai/gpt-oss-20b/blob/main/config.json)
- 核对日期：2026-07-27。

## 常见误解

- **“`num_attention_heads` 就是所有 Q/K/V Head 的统一数量。”** GQA、MQA 中 K/V 数量由另一字段描述。
- **“有 8 个 KV Head 就是 MHA。”** 还要与 Query Head 数量比较。
- **“`head_dim × num_attention_heads` 永远等于 `hidden_size`。”** `gpt-oss-20b` 官方字段就是反例。
- **“公开模型使用 GQA，所以同公司闭源模型也一定使用 GQA。”** 没有公开证据就不能外推。
- **“配置证明了运行一定有某个速度。”** 真实性能还依赖实现、硬件和工作负载。

## 理解检查

1. `num_attention_heads=32`、`num_key_value_heads=32` 通常表示哪种结构？
2. `num_attention_heads=32`、`num_key_value_heads=1` 呢？
3. Qwen3-8B 中一个 KV Head 对应几个 Query Head？
4. 为什么不能把 `hidden_size = num_attention_heads × head_dim` 当成绝对规则？
5. 为什么不能用 `gpt-oss` 配置推断所有闭源 GPT 模型？

返回：[[MHA-GQA与MQA概览|MHA、GQA 与 MQA]]。下一专题：[[MLA与注意力变体概览|MLA 与注意力变体（扩展）]]。
