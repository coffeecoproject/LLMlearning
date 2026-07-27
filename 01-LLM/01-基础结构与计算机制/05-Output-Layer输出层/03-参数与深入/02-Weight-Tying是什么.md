---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer参数与深入概览|Output-Layer参数与深入概览]]"
previous: "[[01-输出形状与LM-Head参数量|输出形状与LM-Head参数量]]"
next: "[[03-Logits到概率的小数字示例|Logits到概率的小数字示例]]"
tags: [llm, weight-tying, embedding, lm-head]
---

# Weight Tying 是什么

> [!summary]
> Weight Tying 指输入 Token Embedding 与输出 LM Head 复用同一份权重参数，而不是各自保存两份独立矩阵。

## 先回忆两端的职责

```text
输入端 Embedding
Token ID → 查到一个 hidden_size 维向量

输出端 LM Head
hidden_size 维 Hidden State → 为所有 Token 计算 Logits
```

两端都需要建立 Token 与 `hidden_size` 维空间之间的联系，因此某些模型让它们共享参数。

## 共享与不共享

### Weight Tying 开启

```text
Embedding Matrix ─┬─ 用于输入查表
                  └─ 转置或等价方式用于输出投影
```

优点之一是减少独立参数，并让输入与输出 Token 表示使用同一组基础权重。

### Weight Tying 关闭

```text
输入 Embedding 权重：一份
输出 LM Head 权重：另一份
```

两组参数可以分别学习适合输入与输出的表示，但参数量更大。

## `tie_word_embeddings` 怎样读

许多模型配置使用：

```json
"tie_word_embeddings": true
```

表示设计上启用共享；`false` 表示通常使用独立权重。但最终仍应结合模型实现和权重文件检查，不能只根据字段名推断所有细节。

## 共享不是可逆

即使共享同一个矩阵：

- Embedding 是按 Token ID 取一行向量；
- LM Head 是用当前 Hidden State 与所有 Token 方向计算匹配分数；
- Transformer 已经在两者之间进行了许多层非线性处理。

所以不能说“输出层把 Embedding 原样倒放一遍”，也不能保证从输出向量唯一还原输入 Token。

## 参数量直觉

设：

```text
vocab_size = V
hidden_size = H
```

若不共享，两端各自可能拥有约 `V×H` 个参数；若共享，就只保留一份主要矩阵。实际模型还包含其他层和可能的 Bias，不能用这一项代表全部参数。

## 开放模型例子

Qwen3-8B 官方配置中：

```json
"tie_word_embeddings": false
```

这表示该版本的输入 Embedding 与输出 LM Head 不采用普通的权重共享设置。它是一个具体模型选择，不能推广成“Qwen 系列或所有 Decoder-only 模型都不共享”。

来源：[Qwen3-8B 官方 `config.json`](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)，核对日期：2026-07-27。

## 理解检查

1. Weight Tying 共享的是计算结果，还是可学习权重？
2. 开启共享后，Embedding 与 LM Head 的操作为什么仍不同？
3. 某个模型配置为 `false`，能否推出所有同系列模型都为 `false`？

下一篇：[[03-Logits到概率的小数字示例|Logits 到概率的小数字示例]]。
