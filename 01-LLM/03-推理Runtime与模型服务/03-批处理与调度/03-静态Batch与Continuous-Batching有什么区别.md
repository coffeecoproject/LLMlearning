---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-批处理与调度概览|批处理与调度概览]]"
previous: "[[02-模型Batch维度与服务端请求Batch有什么区别|模型Batch维度与服务端请求Batch有什么区别]]"
next: "[[04-Prefill与Decode为什么需要区别调度|Prefill与Decode为什么需要区别调度]]"
tags: [llm, runtime, static-batching, continuous-batching]
---

# 静态 Batch 与 Continuous Batching 有什么区别

> [!summary]
> 静态 Batch 倾向于让一组任务一起开始并等待这一组结束；Continuous Batching 会在模型执行轮次之间让请求动态加入和退出。

## 静态Batch的问题

假设 A 需要生成 100 个 Token，B 只需要 5 个。如果 Batch 成员必须固定到整批结束：

```text
A：继续、继续、继续……
B：5步后已经完成，但批次位置可能继续空等A
C：虽然已到达，却要等整个Batch结束
```

这在离线处理相近长度的数据时可能尚可，但不适合请求随时到达、输出长度不可预知的在线服务。

## Continuous Batching怎样改变它

它把“批次”从固定队伍变成每轮可调整的活跃集合：

```text
第1轮：A + B
第2轮：A + B + C
第3轮：A + C      （B完成并退出）
第4轮：A + C + D  （D加入）
```

每轮之后，Runtime 检查谁完成、谁到达、还剩多少 Token 和缓存容量，再组织下一轮。

## 什么没有改变

- A、B、C 的上下文仍互相隔离；
- 单个请求仍遵守自回归因果顺序；
- 模型权重没有因为动态调度而改变；
- Continuous Batching 不保证每个请求延迟都下降，它主要让设备更少空闲、服务更灵活。

## 与训练Batch的区别

训练 Batch 是为了用一组训练样本计算损失并更新参数；这里的 Batch 发生在运行阶段，只组织已经训练好的模型如何处理请求，不做参数更新。

## 理解检查

1. 为什么输出长度未知会让固定 Batch 浪费资源？
2. Continuous Batching 改变了请求成员，为什么没有破坏单请求因果顺序？

下一篇：[[04-Prefill与Decode为什么需要区别调度|Prefill 与 Decode 为什么需要区别调度]]。
