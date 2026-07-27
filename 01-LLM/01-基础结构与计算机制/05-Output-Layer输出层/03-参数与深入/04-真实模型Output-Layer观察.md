---
type: reference
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer参数与深入概览|Output-Layer参数与深入概览]]"
previous: "[[03-Logits到概率的小数字示例|Logits到概率的小数字示例]]"
next: "[[00-Output-Layer边界与复习概览|Output-Layer边界与复习概览]]"
tags: [llm, output-layer, qwen3, deepseek-v3, evidence]
---

# 真实模型 Output Layer 观察

> [!summary]
> 真实开放权重模型会把 Final Norm、LM Head、词表大小和权重共享选择落实为可检查的配置字段或权重名称；这能用来验证抽象结构，但不能据此推断闭源模型的未公开细节。

## Qwen3-8B：从配置连接输入与输出尺寸

Qwen3-8B 官方配置列出：

```json
{
  "hidden_size": 4096,
  "vocab_size": 151936,
  "tie_word_embeddings": false
}
```

可以得到以下已发布事实：

- Transformer 主干 Hidden State 宽度为 4096；
- LM Head 面向 151936 大小的词表空间；
- 该版本没有采用普通输入输出权重共享设置。

由尺寸关系可以进行教学估算：若独立 LM Head 采用无 Bias 的普通线性权重，主要权重规模是：

```text
151936 × 4096 = 622,329,856
```

这是根据配置尺寸得到的结构估算，不等于对整个模型参数量的重新统计。

来源：[Qwen3-8B 官方 `config.json`](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)，核对日期：2026-07-27。

## DeepSeek-V3：从权重结构确认出口

DeepSeek-V3 官方权重说明在 Main Model 的 Output Layer 下明确列出：

```text
model.norm.weight
lm_head.weight
```

它对应当前学习路线中的：

```text
Final Hidden States
→ Final Norm
→ LM Head
```

同一说明还指出，主模型的输入 Embedding 和输出 Head 各约包含 0.9B 激活参数；额外 MTP 模块会共享主模型输出 Head。MTP 是模型扩展结构，不应反过来干扰对普通主输出层的基础理解。

来源：[DeepSeek-V3 官方权重说明](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/README_WEIGHTS.md)，核对日期：2026-07-27。

## 可以确认与不能确认的内容

| 类型 | 结论 |
|---|---|
| 已发布事实 | 上述具体版本配置与权重说明中的字段、尺寸和名称 |
| 合理结构解释 | `model.norm.weight → lm_head.weight` 对应 Final Norm 与词表投影 |
| 不能外推 | OpenAI 闭源模型或其他版本一定采用相同尺寸、共享策略或输出变体 |

## 为什么学习真实配置

目的不是背参数，而是练习把抽象概念映射到真实资产：

```text
hidden_size → LM Head 输入宽度
vocab_size → LM Head 输出宽度
tie_word_embeddings → 是否共享输入输出权重
model.norm.weight → Final Norm 参数
lm_head.weight → 输出投影参数
```

## 理解检查

1. Qwen3-8B 的 `vocab_size` 怎样影响 LM Head？
2. `tie_word_embeddings=false` 能否推出模型没有输入 Embedding？
3. 为什么不能用 DeepSeek-V3 的公开权重结构推断 GPT 闭源模型的具体实现？

继续阅读：[[00-Output-Layer边界与复习概览|Output Layer 边界与复习]]。
