---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-单请求效率与资源概览|单请求效率与资源概览]]"
previous: "[[02-输出长度为什么直接增加Decode轮数|输出长度为什么直接增加Decode轮数]]"
next: "[[04-模型权重临时激活与KV-Cache分别占用什么资源|模型权重、临时激活与KV-Cache分别占用什么资源]]"
tags: [llm, inference, ttft, tpot, latency, tokens-per-second]
---

# TTFT、TPOT 与总延迟怎样区分

> [!summary]
> TTFT 描述多久看见第一个输出 Token，TPOT 描述第一个 Token 之后平均隔多久产生一个新 Token，总延迟描述完整回答何时结束；它们衡量的是不同体验，不能互相替代。

> [!phase] 两个观察边界
> 本专题主要解释请求内部计算；真实 API 从客户端测到的数字还可能包括网络传输、排队、调度和服务封装。看到性能报告时，必须先确认测量起点和终点。

## TTFT：第一个Token要等多久

TTFT 是 **Time To First Token**，即从请求开始到第一个输出 Token 可用的时间。

从模型主线看，它主要覆盖：

```text
输入准备
→ Prefill全部输入
→ 得到第一组Logits
→ 选择第一个Token
```

从远程用户实际观察看，还可能额外包含：

```text
客户端发送
+ 网络传输
+ 服务排队
+ Runtime调度
+ 请求内部计算
+ 首段返回传输
```

因此，TTFT 高不一定只说明 Prefill 慢；必须知道指标是本地模型测量还是端到端服务测量。

## TPOT：后续Token之间平均等多久

TPOT 是 **Time Per Output Token**。常见口径是第一个 Token 产生之后，后续输出 Token 平均需要多少时间。

例如：

```text
后续平均每100 ms产生一个Token
→ TPOT约为100 ms/token
```

TPOT 越小，用户通常感觉文字“吐得越快”。

## Tokens/s：TPOT的另一种表达

在同一测量范围内，可以近似转换：

```text
Tokens/s ≈ 1 ÷ TPOT（秒/token）
```

例如：

```text
TPOT = 0.1秒/token
Tokens/s ≈ 10 token/s
```

但“Tokens/s”必须说明是：

- 单个请求的输出速度；
- 多请求合计的服务吞吐量；
- 只统计 Decode，还是包含其他阶段。

名称相同，口径可能完全不同。

## 总延迟：完整回答何时结束

如果输出总数为 `N`，并把第一个 Token 算入 TTFT，可以用一个教学近似：

```text
总延迟 ≈ TTFT + (N - 1) × TPOT
```

示例：

```text
TTFT：0.8秒
总输出：5 Token
TPOT：0.1秒/token

总延迟 ≈ 0.8 + 4 × 0.1
        ≈ 1.2秒
```

> [!note] 公式边界
> 这是平均估算，不是硬性定律。每轮 Decode 时间可能随历史长度变化，网络可能分块返回，Tokenizer Decode 和客户端渲染也可能引入额外时间。

## 两种体验为什么可能相反

假设两个系统：

| 系统 | TTFT | TPOT | 直观体验 |
|---|---:|---:|---|
| A | 0.3 秒 | 0.2 秒/token | 很快开始，但后面慢 |
| B | 1.0 秒 | 0.05 秒/token | 开始较慢，但后面快 |

短回答可能 A 更快结束；长回答可能 B 后来居上。因此，不能只用 TTFT 或只用 Tokens/s 宣称一个系统“更快”。

## Streaming改变的是哪个时间

Streaming 让客户端在回答尚未完成时逐步看到内容，因此主要改善**感知等待体验**：

```text
非流式：等完整答案后一次显示
流式：首段到达后逐步显示
```

Streaming 不必然减少模型自身的 Prefill 或 Decode 计算。网络传输与客户端显示机制会在 Runtime 专题继续讨论。

## 常见误解

1. **“TTFT就是Prefill时间。”** 本地细分测量中二者密切相关；端到端 TTFT 还可能包含网络和排队。
2. **“Tokens/s越高，总回答一定越快。”** 还要看 TTFT 和输出长度。
3. **“Streaming让模型计算更快。”** 它主要让已生成内容更早可见，不必然改变模型计算速度。
4. **“所有性能报告的Tokens/s都能直接比较。”** 单请求与全服务、输入与输出、硬件和测试条件必须一致。

## 理解检查

1. 为什么短回答中 TTFT 可能比 TPOT 更影响体验？
2. 为什么同样写着 100 Tokens/s，两个报告可能不是同一指标？
3. Streaming 为什么能改善感知体验，却不一定减少总计算量？

下一篇：[[04-模型权重临时激活与KV-Cache分别占用什么资源|模型权重、临时激活与 KV Cache 分别占用什么资源]]。
