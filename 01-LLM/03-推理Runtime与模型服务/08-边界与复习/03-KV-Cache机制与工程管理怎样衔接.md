---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-推理Runtime边界与复习概览|推理Runtime边界与复习概览]]"
previous: "[[02-服务启动单请求与多请求服务状态怎样区分|服务启动单请求与多请求服务状态怎样区分]]"
next: "[[04-单请求自回归顺序为什么不被多请求并发改变|单请求自回归顺序为什么不被多请求并发改变]]"
tags: [llm, runtime, kv-cache, mechanism, memory-management]
---

# KV Cache 机制与工程管理怎样衔接

> [!summary]
> KV Cache 机制解释为什么历史 Attention 状态可以复用；工程管理解释许多请求产生的缓存怎样在有限内存中分配、定位、共享合格前缀并安全回收。

> [!phase] 运行阶段：机制与系统衔接

## 机制层回答“为什么能缓存”

在 Causal Attention 中，过去位置形成的 K、V 不会因为未来又出现一个 Token 而被改写。因此新 Token 可以读取已经保存的历史状态：

```text
过去Token的K/V已经算过
→ 保存并复用
→ 新Token只追加自己的新状态
```

这是模型生成机制，入口是 [[01-KV-Cache保存什么|KV Cache 保存什么]]。

## 工程层回答“缓存放在哪里”

模型服务同时处理许多请求后，还必须回答：

```text
A的缓存使用哪些Block？
B继续生成时还有没有空间？
C能否复用某段完全匹配的前缀？
A结束后哪些Block可以回收？
缓存池不足时谁等待？
```

这些属于 [[00-KV-Cache工程管理概览|KV Cache 工程管理]]。

## 两层怎样在一次Decode中连接

```text
调度器选择请求A参与本轮Decode
→ KV Cache Manager找到A的历史Block
→ 当前层Attention读取A自己的历史状态
→ 模型得到新的Hidden State和Logits
→ 当前Token的新K/V追加到A的缓存
→ 更新A的长度与缓存映射
```

如果没有模型机制，Runtime 没有可复用的 K、V；如果没有工程管理，多请求服务就难以安全高效地保存和查找这些状态。

## Prefix Cache为什么仍是工程复用

Prefix Cache 复用的是已经计算完成、条件匹配且允许共享的前缀状态。它没有改变 Attention 的基本公式，也没有把缓存训练进模型参数。

```text
相同前缀再次出现
→ Runtime找到合格缓存块
→ 减少重复Prefill
```

## 三种“复用”不要混淆

| 复用对象 | 发生在哪里 | 作用 |
|---|---|---|
| 单请求历史KV | 同一次生成的多轮Decode | 避免反复生成历史K/V |
| 跨请求Prefix Cache | Runtime缓存池 | 避免重复前缀Prefill |
| 模型权重 | 多个请求共同调用同一模型 | 避免每请求复制完整参数 |

它们复用的是不同对象，生命周期也不同。

## 常见误解

1. **“PagedAttention创造了KV Cache机制。”** KV Cache机制来自自回归Attention的可复用状态；PagedAttention解决工程组织与访问。
2. **“Prefix Cache是模型学会了记住用户。”** 它是运行时计算复用，不更新参数。
3. **“请求共享模型权重，所以也共享同一份KV内容。”** 权重共享与请求状态隔离可以同时成立。

## 理解检查

1. KV Cache机制与KV Cache Manager分别回答什么问题？
2. 一轮Decode中，调度器、Cache Manager和Attention怎样依次协作？
3. 单请求复用与Prefix Cache复用有什么区别？

下一篇：[[04-单请求自回归顺序为什么不被多请求并发改变|单请求自回归顺序为什么不被多请求并发改变]]。
