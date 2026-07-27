---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-单请求效率与资源概览|单请求效率与资源概览]]"
previous: "[[01-输入长度为什么主要影响Prefill|输入长度为什么主要影响Prefill]]"
next: "[[03-TTFT-TPOT与总延迟怎样区分|TTFT、TPOT与总延迟怎样区分]]"
tags: [llm, inference, decode, output-length, autoregressive]
---

# 输出长度为什么直接增加 Decode 轮数

> [!summary]
> 普通自回归生成中，下一 Token 的分布依赖已经实际选出的前缀；因此多生成一个 Token，通常就要再完成一轮模型 Decode、Token 选择和缓存更新。

> [!phase] 运行阶段
> 本篇描述普通 Decoder-only 自回归生成的逻辑主线。Speculative Decoding 等 Runtime 优化可以一次验证多个候选，但不会取消最终序列的条件依赖。

## 为什么输出不能像Prefill那样一次确定

Prefill 面对的是已经知道的完整输入。Decode 面对的未来 Token 尚未确定：

```text
先选出Token 1
→ 才能计算 P(Token 2 | 输入, Token 1)
→ 选出Token 2
→ 才能计算 P(Token 3 | 输入, Token 1, Token 2)
```

如果第一个 Token 选成不同内容，后面的概率分布也可能随之改变。所以普通生成不能先把未来所有 Token 当成已知输入，一次性完成相同的 Prefill 计算。

## 每多一个输出Token增加什么

一轮常规 Decode 大致包含：

```text
刚选出的Token ID
→ Embedding和当前位置
→ 经过全部Transformer Blocks
→ 当前Query读取历史KV
→ 当前K、V加入KV Cache
→ Output Layer产生新Logits
→ 生成控制器选择下一个Token
```

因此，输出从 20 Token 增长到 100 Token，通常意味着多出约 80 轮这样的顺序过程。

## 一个简化时间例子

假设教学场景中：

```text
产生首Token前等待：800 ms
后续每个Token平均：100 ms
```

如果总共生成 5 个输出 Token：

```text
首Token已经包含在800 ms中
剩余4个Token约需 4 × 100 ms
总时间约为 1,200 ms
```

如果生成 25 个 Token：

```text
800 ms + 24 × 100 ms
≈ 3,200 ms
```

这是便于理解的近似。真实每轮时间可能变化，并不保证完全相同。

## 为什么越到后面每轮可能略有变化

Decode 虽然只为新增位置计算新的表示，但当前 Query 仍要读取可见历史 KV：

```text
第1轮：输入历史
第2轮：输入历史 + 输出1
第3轮：输入历史 + 输出1 + 输出2
……
```

历史持续变长，KV Cache 也持续增长。因此“每 Token 时间”常用平均值表示，不必假设整段回答中每轮完全一样快。

## KV Cache节省了什么

KV Cache 避免每一轮重新计算全部历史 Token 的 K、V：

```text
没有缓存的直观做法
每轮重新处理输入 + 全部已生成Token

使用KV Cache
只为新增Token计算新状态
+ 读取历史已缓存状态
```

它大幅减少重复计算，但没有让未来 Token 失去先后依赖，也没有让历史上下文消失。

## Speculative Decoding改变了什么

某些 Runtime 可以让较小模型或其他方法先猜一段候选，再由目标模型批量验证。用户可能观察到一次推进多个 Token。

但其逻辑仍然是：

```text
候选Token可以提前猜
→ 目标模型必须验证候选是否符合逐步条件
→ 不符合的位置之后需要重新生成
```

因此，本篇使用“一轮普通 Decode 对应推进一个确定 Token”的心智模型仍然成立；Speculative Decoding 属于额外运行优化。

## 常见误解

1. **“Streaming每次显示一个Token，所以模型只能一次计算一个字符。”** Token 不是字符；客户端也可能把多个 Token 合并成一个显示片段。
2. **“GPU擅长并行，所以未来100个Token可以普通地同时生成。”** 一轮内部可以并行，未来轮次之间仍有条件依赖。
3. **“有KV Cache后输出长度不再影响时间。”** 缓存减少重复工作，但更多输出仍意味着更多 Decode 轮次。
4. **“设置最大输出1000 Token就一定生成1000 Token。”** EOS、停止序列或其他停止条件可能更早结束。

## 理解检查

1. 为什么第 3 个输出 Token 需要等待前两个 Token 被实际选定？
2. KV Cache 为什么不能消除 Decode 的顺序轮数？
3. Speculative Decoding 为什么是优化，而不是改变自回归条件依赖？

下一篇：[[03-TTFT-TPOT与总延迟怎样区分|TTFT、TPOT 与总延迟怎样区分]]。
