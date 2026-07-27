---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-单请求生成主线概览|单请求生成主线概览]]"
previous: "[[02-为什么读取最后有效位置的Logits|为什么读取最后有效位置的Logits]]"
next: "[[01-KV-Cache保存什么|KV-Cache保存什么]]"
tags: [llm, inference, kv-cache, decode]
---

# KV Cache 与增量 Decode 概览

> [!summary]
> KV Cache 保存此前 Token 在各 Attention 层形成的可复用状态；Decode 把新 Token 送过模型，在 Attention 中复用历史状态，最后经 Output Layer 得到下一轮 Logits。

## 三者怎样连接

```text
Prefill
→ 为输入上下文建立 KV Cache
→ 选出第一个新 Token
→ 新 Token 经过 Embedding
→ 经过全部 Transformer Blocks
   └─ 各层 Attention 读取历史 K/V，并追加当前 Token 的新 K/V
→ 经过 Output Layer 得到下一轮 Logits
→ 再选一个 Token
→ 重复
```

因此，“增量”表示历史 Attention 状态可以复用，不表示新 Token 只经过 KV Cache，也不表示它跳过了模型的其他部分。

## 阅读顺序

1. [[01-KV-Cache保存什么|KV Cache 保存什么]]；
2. [[02-Decode怎样增量生成|Decode 怎样增量生成]]；
3. [[03-为什么运行时仍然不能看未来|为什么运行时仍然不能看未来]]。

## 先划清边界

KV Cache 是单请求增量生成的核心机制；缓存怎样分页、跨请求分配和调度属于 [[00-KV-Cache工程管理概览|Runtime 的 KV Cache 工程管理]]。

下一篇：[[01-KV-Cache保存什么|KV Cache 保存什么]]。
