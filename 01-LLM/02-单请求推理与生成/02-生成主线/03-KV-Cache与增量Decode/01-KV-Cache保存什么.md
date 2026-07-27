---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-KV-Cache与增量Decode概览|KV-Cache与增量Decode概览]]"
previous: "[[00-KV-Cache与增量Decode概览|KV-Cache与增量Decode概览]]"
next: "[[02-Decode怎样增量生成|Decode怎样增量生成]]"
tags: [llm, inference, kv-cache, attention, qkv]
---

# KV Cache 保存什么

> [!summary]
> 标准 Attention 的 KV Cache 保存此前 Token 在每一层 Attention 中已经计算出的 Key 和 Value；新 Token 可以直接读取它们，不必为历史 Token 重复生成相同的 K、V。

## 为什么过去的 K、V 可以复用

在 Decoder-only Causal Attention 中，一个历史 Token 不能看到它之后才出现的未来 Token。因此，当未来又生成一个新 Token 时，过去位置已经形成的 K、V 不需要因这个新 Token 而改写。

例如已经处理：

```text
[我] [喜欢] [猫]
```

准备生成下一个 Token 时，模型需要让当前 Query 与历史 Key 匹配，并从历史 Value 中取回信息。历史 `[我][喜欢][猫]` 的 K、V 已经算过，可以缓存复用。

## Cache 按什么结构存在

概念上，每个 Transformer Attention 层都有自己的历史状态：

```text
Layer 1：过去位置的 K、V
Layer 2：过去位置的 K、V
……
Layer N：过去位置的 K、V
```

因为不同层的输入 Hidden States 不同，它们投影出的 K、V 也不同，不能只存一份给所有层共用。

## 它不保存什么

- 不是模型权重；
- 不是对话的永久记忆；
- 不是整个互联网或知识库；
- 标准 KV Cache 不是简单保存“原始文字”；
- 通常也不是把所有历史 Hidden State 原封不动保存下来。

## 为什么名字不能推广到所有架构细节

标准 MHA、GQA、MQA 通常都可以用“缓存 K、V”理解，但不同结构的 KV 头数和形状不同。DeepSeek-V3 的 MLA 会缓存更紧凑的潜在表示并在计算中恢复所需信息，因此具体存储形式不能只凭“KV Cache”这个通用名称推断。

## 开放实现证据

Hugging Face Transformers 官方文档把 Cache 组织为按层保存 Key 和 Value 的结构，并说明它用于复用自回归生成中的历史计算。`Qwen/Qwen3-8B` 官方配置公开 `use_cache: true`，说明其标准运行配置支持缓存，但具体 Runtime 的内存布局仍由实现决定。

来源：[Hugging Face Caching 官方文档](https://huggingface.co/docs/transformers/cache_explanation)、[Qwen3-8B config.json](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)，核对日期：2026-07-27。

## 理解检查

1. 为什么每层都需要自己的缓存？
2. 为什么未来 Token 不会迫使历史位置重新生成 K、V？
3. KV Cache 为什么不等于长期记忆？

下一篇：[[02-Decode怎样增量生成|Decode 怎样增量生成]]。
