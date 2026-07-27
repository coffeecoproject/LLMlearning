---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-单请求运行与生成概览|单请求运行与生成概览]]"
previous: "[[02-为什么读取最后有效位置的Logits|为什么读取最后有效位置的Logits]]"
next: "[[01-KV-Cache保存什么|KV-Cache保存什么]]"
tags: [llm, inference, kv-cache, decode]
---

# KV Cache 与增量 Decode 概览

> [!summary]
> KV Cache 保存此前 Token 在各 Attention 层形成的可复用状态；Decode 每次只为新 Token 计算新增状态，再读取历史缓存完成下一步预测。

## 三者怎样连接

```text
Prefill
→ 为输入上下文建立 KV Cache
→ 选出第一个新 Token
→ Decode 这个新 Token，并把新 K/V 追加到 Cache
→ 再选一个 Token
→ 重复
```

## 阅读顺序

1. [[01-KV-Cache保存什么|KV Cache 保存什么]]；
2. [[02-Decode怎样增量生成|Decode 怎样增量生成]]；
3. [[03-为什么运行时仍然不能看未来|为什么运行时仍然不能看未来]]。

## 先划清边界

KV Cache 是单请求增量生成的核心机制；缓存怎样分页、跨请求分配和调度属于 [[00-Runtime服务系统概览|Runtime 服务系统]]。

下一篇：[[01-KV-Cache保存什么|KV Cache 保存什么]]。
