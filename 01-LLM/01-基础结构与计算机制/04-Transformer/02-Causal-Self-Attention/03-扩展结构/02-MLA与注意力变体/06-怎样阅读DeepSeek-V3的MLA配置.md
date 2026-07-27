---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-MLA与注意力变体概览|MLA与注意力变体概览]]"
previous: "[[05-MLA与MHA-GQA-MQA有什么不同|MLA与MHA-GQA-MQA有什么不同]]"
next: "[[07-其他注意力变体地图|其他注意力变体地图]]"
tags: [llm, mla, deepseek-v3, model-config, open-weight-model]
---

# 怎样阅读 DeepSeek-V3 的 MLA 配置

> [!summary]
> DeepSeek-V3 的 MLA 不能只用 Head 数量判断；需要一起读取 Query/KV 低秩宽度、内容 Q/K 宽度、RoPE Q/K 宽度和 Value 宽度，再用官方模型代码确认这些字段怎样连接。

## 官方配置中的关键字段

DeepSeek-V3 官方 `config.json` 公布：

```json
{
  "hidden_size": 7168,
  "num_attention_heads": 128,
  "num_key_value_heads": 128,
  "q_lora_rank": 1536,
  "kv_lora_rank": 512,
  "qk_nope_head_dim": 128,
  "qk_rope_head_dim": 64,
  "v_head_dim": 128
}
```

这些是对应公开版本的配置事实。下面会明确区分“字段直接公布了什么”和“我们由此做出的结构推导”。

## `hidden_size = 7168`

它表示 Transformer 主 Hidden State 的宽度：

```text
每个 Token 进入该层 MLA 前
→ 主表示宽度为 7168
```

它不是 KV Latent 的宽度，也不是单个 Head 的宽度。

## `num_attention_heads = 128`

官方 MLA 实现把 Query 展开为 128 个 Head。每个 Query Head 都可产生自己的匹配 Score、Weight 和 Context 路径。

`num_key_value_heads = 128` 也出现在配置中，但不能脱离自定义 MLA 实现，直接套用普通 GQA 判断规则。关键证据还包括后面的低秩与分量字段。

## `q_lora_rank = 1536`

它表示 Query 向下投影的中间宽度：

```text
7168 维 Hidden State
→ 1536 维 Query Latent
→ 多个 Query Head
```

`rank = 1536` 不表示有 1536 个 Query Head，也不表示 Query 只能看 1536 个 Token。

这里的 `lora` 是基础 MLA 架构内置的 low-rank projection，不表示在模型外额外安装了用于微调的 LoRA Adapter。

## `kv_lora_rank = 512`

它表示联合 KV 内容 Latent 的宽度：

```text
7168 维 Hidden State
→ 512 维 KV Latent
→ 提供内容 Key 与 Value 信息
```

这 512 维是 MLA 压缩路径的核心字段。它不是 `vocab_size`，也不是上下文窗口长度。

## 两种 Q/K Head 分量

```text
qk_nope_head_dim = 128
→ 每个 Q/K Head 中不直接应用 RoPE 的内容分量宽度

qk_rope_head_dim = 64
→ 每个 Query Head 的 RoPE 分量宽度，也是逐 Token 共享位置 Key 的宽度
```

因此可以推导：

```text
每个 Query Head 与其使用的有效 Key 的匹配宽度
= 128 内容维度 + 64 RoPE 维度
= 192
```

这个 `192` 是根据两个公开字段相加得到的推导值，不是配置中名为 `head_dim` 的原始字段。

## `v_head_dim = 128`

它表示每个 Value Head 的宽度。Value 用于 Weight 之后的信息汇总：

```text
每个 Query Head 的 Weight
× 对应 Value 信息
→ 128 维 Head Context
```

128 个 Head 的 Context 随后合并，并通过 Output Projection 回到主 Hidden State 宽度。

## 把真实配置串起来

```text
输入：7168 维 Hidden State

Query 路径：
7168 → 1536 Query Latent
→ 128 个 Query Head
→ 每个 Head = 128 内容维 + 64 RoPE 维

KV 路径：
7168 → 512 KV Latent
→ 为 128 个 Head 提供内容 K 与 128 维 Value

位置 Key：
另有逐 Token 的 64 维 RoPE Key 分量，由各 Head 共享
```

## 官方代码进一步告诉了什么

DeepSeek-V3 官方推理实现展示了两条等价目标、不同中间张量的路径：

```text
naive 路径
→ 显式展开并保存完整多头 K/V

优化路径
→ 保存 512 维 kv_cache
+ 64 维 pe_cache
→ 利用矩阵吸收完成匹配与 Value 汇总
```

因此，在该实现的优化路径中，可以把每个历史位置、每层保存的 MLA 核心宽度先理解为：

```text
512 维内容 Latent + 64 维位置 Key
```

这只是张量宽度关系，不是完整显存数字。总显存还要乘以层数、序列长度、Batch、数据类型，并考虑缓存管理和实现开销，留到普通运行模块计算。

## 直接事实、推导与未知信息

### 官方直接公开

- 上述配置字段及数值；
- DeepSeek-V3 采用 MLA；
- 官方代码存在 `naive` 与优化 Attention 路径；
- 优化路径定义了 `kv_cache` 与 `pe_cache`。

### 根据字段和代码推导

- Query/Key 匹配宽度为 `128 + 64 = 192`；
- 优化路径核心历史表示宽度为 `512 + 64 = 576`；
- 仅比较两个 Head 数量不足以识别该自定义 MLA。

### 不能只凭配置确定

- 每个维度实际编码了什么人类可读语义；
- 在所有框架和硬件上的真实速度；
- 某次 API 请求具体使用哪个运行内核；
- MLA 单独贡献了多少最终回答质量。

## 官方来源

- [DeepSeek-V3 官方 `config.json`](https://huggingface.co/deepseek-ai/DeepSeek-V3/blob/main/config.json)
- [DeepSeek-V3 官方推理实现 `model.py`](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/inference/model.py)
- [DeepSeek-V3 官方技术报告](https://arxiv.org/abs/2412.19437)
- 核对日期：2026-07-27。

## 常见误解

- **“`q_lora_rank=1536` 表示 1536 个 Query Head。”** Head 数量是 128，1536 是 Query Latent 宽度。
- **“字段带 `lora`，说明这是外加的 LoRA 微调 Adapter。”** 这里是 DeepSeek-V3 基础 MLA 结构自身的低秩投影。
- **“`kv_lora_rank=512` 表示最多缓存 512 个 Token。”** 它是每个位置的内容 Latent 宽度。
- **“`num_attention_heads` 与 `num_key_value_heads` 相等，所以一定是普通 MHA。”** 自定义 MLA 字段与实现否定了这种简单判断。
- **“每个 Q/K 匹配宽度就是 128。”** 匹配还包含 64 维 RoPE 分量，总计 192；其中位置 Key 在 Head 之间共享。
- **“576 就是总 KV Cache 大小。”** 它只是每位置、每层的核心元素宽度，尚未乘其他维度与数据类型。
- **“官方代码有 naive 路径，所以 MLA 不能减少历史保存量。”** 优化路径正是围绕 Latent 与位置分量保存。

## 理解检查

1. `q_lora_rank` 与 Query Head 数量分别是多少？
2. `kv_lora_rank` 描述什么，为什么不是上下文长度？
3. DeepSeek-V3 每个 Query Head 与有效 Key 的匹配宽度怎样得到 192？
4. 为什么 `512 + 64 = 576` 还不是完整 KV Cache 显存大小？
5. 为什么读取自定义模型时必须同时看配置和实现？

下一篇：[[07-其他注意力变体地图|其他注意力变体地图]]。
