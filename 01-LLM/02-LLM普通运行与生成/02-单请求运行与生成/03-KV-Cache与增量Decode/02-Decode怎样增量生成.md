---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-KV-Cache与增量Decode概览|KV-Cache与增量Decode概览]]"
previous: "[[01-KV-Cache保存什么|KV-Cache保存什么]]"
next: "[[03-为什么运行时仍然不能看未来|为什么运行时仍然不能看未来]]"
tags: [llm, inference, decode, autoregressive, kv-cache]
---

# Decode 怎样增量生成

> [!summary]
> Decode 阶段每轮接收刚选出的新 Token，计算它在各层的新表示并读取历史 KV Cache，最后产生下一轮 Logits。

## 一轮 Decode 发生什么

假设 Prefill 已处理 Prompt，并选出了回答的第一个 Token：

```text
新 Token ID
→ Embedding 与当前位置处理
→ 逐层 Transformer 计算
   当前 Query 读取历史 K/V
   计算当前 Token 的新 K/V 并加入 Cache
→ Output Layer
→ 新一组 Logits
→ 再选择一个 Token
```

## 为什么叫“增量”

没有缓存时，最直观的做法是把“Prompt + 已生成全部 Token”每轮重新送入模型。KV Cache 让运行时只需为新增 Token 形成新的 Attention 状态，同时复用历史状态。

这减少的是**重复计算**，不是让模型跳过历史上下文。新 Token 的 Query 仍会与可见历史状态发生 Attention。

## 一个四轮示例

```text
Prompt：法国的首都是

Prefill → 选“巴”
Decode“巴” → 选“黎”
Decode“黎” → 选“。”
Decode“。” → 选 <EOS>
停止
```

真实 Token 切分可能把“巴黎”作为一个 Token，也可能不是；这里是虚构示例，只展示循环依赖。

## 为什么不能普通地并行生成未来十个 Token

要知道第 2 个新 Token 的分布，必须先知道第 1 个新 Token 实际选了什么；第 3 个又依赖前两个。硬件可以并行执行一轮内部矩阵计算，也可以并行处理多个请求，但普通自回归链的未来步骤存在先后依赖。

某些 Runtime 使用 Speculative Decoding 等方法猜测并批量验证候选 Token，但那是额外优化，并没有改变“最终序列必须满足逐步条件依赖”的基本事实。

## Decode 与 Tokenizer Decode 不是一回事

- **模型 Decode 阶段**：利用缓存逐步计算新 Token；
- **Tokenizer Decode**：把 Token ID 序列恢复成字节或可显示文字。

中文都常译作“解码”，必须根据上下文区分。

## 理解检查

1. Decode 每一轮的新输入是什么？
2. KV Cache 减少了什么，又没有省略什么？
3. 为什么模型内部可并行计算不等于未来 Token 可同时确定？

下一篇：[[03-为什么运行时仍然不能看未来|为什么运行时仍然不能看未来]]。
