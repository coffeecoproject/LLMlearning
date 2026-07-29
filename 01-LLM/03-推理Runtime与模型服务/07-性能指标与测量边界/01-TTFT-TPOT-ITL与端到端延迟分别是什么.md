---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-性能指标与测量边界概览|性能指标与测量边界概览]]"
previous: "[[00-性能指标与测量边界概览|性能指标与测量边界概览]]"
next: "[[02-单请求速度与服务吞吐量为什么可能冲突|单请求速度与服务吞吐量为什么可能冲突]]"
tags: [llm, runtime, ttft, tpot, itl, latency]
---

# TTFT、TPOT、ITL 与端到端延迟分别是什么

> [!summary]
> TTFT 衡量等到第一个 Token 的时间，ITL 衡量相邻输出 Token 的间隔，TPOT 概括首 Token 之后每个输出 Token 的平均耗时，端到端延迟衡量整项请求多久完成。

## 先看一条时间线

```text
t0 提交请求
│ 排队、准备、Prefill
t1 收到第一个输出Token
│ Decode并逐步返回
t2 收到第二个Token
│ ……
tn 请求完成
```

| 指标 | 它回答什么 | 用户感受 |
|---|---|---|
| TTFT（Time to First Token） | `t0 → t1` 多久 | 点发送后多久开始出现回答 |
| ITL（Inter-Token Latency） | 相邻 Token 之间隔多久 | 输出是否忽快忽慢、出现停顿 |
| TPOT（Time per Output Token） | 首 Token 后平均每个 Token 多久 | 后续整体生成节奏 |
| E2E Latency | `t0 → tn` 多久 | 整项请求多久真正完成 |

## 一个简易数字例子

若第一个 Token 在 0.8 秒出现，之后四个 Token 分别每隔约 0.1 秒到达：

- TTFT 约为 0.8 秒；
- 后续 ITL 约为 0.1 秒；
- TPOT 约为 0.1 秒；
- 五个 Token 的端到端时间约为 1.2 秒。

数字仅用于说明关系，不代表真实模型速度。

## 为什么TPOT与ITL都存在

TPOT 是一段输出的平均概括；ITL 能观察每两个 Token 之间的实际波动。平均值相同的两个请求，一个可能很平稳，另一个可能时快时慢，用户体验并不相同。

不同工具对起点和统计口径可能略有差异，比较前必须看定义。vLLM 的公开指标同时记录 TTFT、相邻 Token 延迟、队列时间、Prefill、Decode 和端到端延迟，说明单一数字无法覆盖完整体验。

来源：[vLLM Metrics](https://docs.vllm.ai/en/stable/design/metrics/)、[vLLM Benchmark CLI](https://docs.vllm.ai/en/stable/benchmarking/cli/)，核对日期：2026-07-28。

## 常见误解

**TTFT 很低，所以整段一定完成得快。** 后续 Decode 很慢或输出很长时，端到端时间仍可能很长。

**网络界面每次刷新就是一个 ITL。** [[05-网络分块为什么不等于模型Token|网络分块不一定对应模型 Token]]，客户端显示还会混入网络和渲染时间。

## 理解检查

1. “很快开始回答，但后面不断停顿”分别对应哪些指标？
2. 为什么输出长度不同的请求不能只比较端到端时间？

下一篇：[[02-单请求速度与服务吞吐量为什么可能冲突|单请求速度与服务吞吐量为什么可能冲突]]。
