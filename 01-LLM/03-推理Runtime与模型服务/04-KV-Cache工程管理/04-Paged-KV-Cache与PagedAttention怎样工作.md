---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-KV-Cache工程管理概览|KV-Cache工程管理概览]]"
previous: "[[03-连续内存分配为什么产生浪费与碎片|连续内存分配为什么产生浪费与碎片]]"
next: "[[05-KV-Cache怎样分配追加释放与驱逐|KV-Cache怎样分配追加释放与驱逐]]"
tags: [llm, runtime, kv-cache, paged-attention, block-table]
---

# Paged KV Cache 与 PagedAttention 怎样工作

> [!summary]
> Paged KV Cache 把请求的历史缓存分散存入固定大小的物理块，并用映射保持逻辑顺序；PagedAttention 让 Attention 能按照这份映射读取非连续的缓存块。

> [!phase] 运行阶段：公开实现概念

## 先区分三个对象

| 对象 | 作用 |
|---|---|
| KV Cache Block | 保存一段 Token 位置对应的缓存状态 |
| Block Table或映射 | 记录某个请求的逻辑顺序对应哪些物理块 |
| PagedAttention | 按映射读取这些块并完成Attention计算 |

“分页管理”和“Attention 怎样读取分页数据”相连，但不是完全相同的词。

## 一个教学例子

假设一个 Block 能容纳 4 个 Token 位置。请求 A 已处理 10 个位置：

```text
逻辑第1段：位置1～4
逻辑第2段：位置5～8
逻辑第3段：位置9～10
```

它们可能实际放在：

```text
逻辑第1段 → 物理Block 7
逻辑第2段 → 物理Block 2
逻辑第3段 → 物理Block 9
```

Runtime 保存映射：

```text
请求A：[7, 2, 9]
```

对请求 A 来说，历史仍按位置 1 到 10 排列；只是物理存储不要求连续。

> [!warning] 示例边界
> Block容量和编号均为虚构，只用于理解逻辑顺序与物理位置的区别。

## 新Token怎样追加

如果最后一个 Block 还有空位，新位置可以继续写入；如果已经填满，Runtime 再从空闲池分配一个 Block，并把编号追加到请求映射。

```text
请求A：[7, 2, 9]
→ Block 9填满
→ 再分配Block 5
→ 请求A：[7, 2, 9, 5]
```

## PagedAttention解决了什么连接问题

普通 Attention 仍需要当前 Query 与历史缓存进行匹配。PagedAttention 的关键是让计算能够根据 Block Table 找到历史 K/V，而不是要求整个请求的缓存物理连续。

它没有改变模型已经学到的参数，也没有取消 Causal Attention；它改变的是 Runtime 怎样组织和访问缓存内存。

## 为什么不能推广成唯一方案

PagedAttention 是 vLLM 公开提出并实现的重要方案，但其他 Runtime 可以采用不同的连续虚拟内存、分页方式、内核和缓存管理设计。

因此正确表达是：

```text
分页是解决动态KV Cache管理的一类工程思路
PagedAttention是其中一个公开实现体系
```

## 来源

- [Efficient Memory Management for Large Language Model Serving with PagedAttention](https://arxiv.org/abs/2309.06180)
- [vLLM Automatic Prefix Caching](https://docs.vllm.ai/en/stable/design/prefix_caching/)

核对日期：2026-07-28。

## 常见误解

1. **“物理块不连续，所以模型看到的上下文顺序也乱了。”** Block Table 保持逻辑顺序。
2. **“PagedAttention是Transformer新增的一层。”** 它是推理执行和缓存访问方案。
3. **“每个Token单独占一个Page。”** Block通常覆盖一段位置，具体大小由实现决定。
4. **“所有Runtime都使用vLLM同样的分页结构。”** 需要查看对应实现证据。

## 理解检查

1. Block、Block Table 和 PagedAttention 分别负责什么？
2. 为什么物理不连续不等于逻辑历史不连续？
3. PagedAttention改变了模型参数还是缓存访问方式？

下一篇：[[05-KV-Cache怎样分配追加释放与驱逐|KV Cache 怎样分配、追加、释放与驱逐]]。
