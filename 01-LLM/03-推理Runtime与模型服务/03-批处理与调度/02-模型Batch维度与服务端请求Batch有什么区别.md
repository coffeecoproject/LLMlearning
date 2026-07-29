---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-批处理与调度概览|批处理与调度概览]]"
previous: "[[01-为什么模型服务需要同时管理多个请求|为什么模型服务需要同时管理多个请求]]"
next: "[[03-静态Batch与Continuous-Batching有什么区别|静态Batch与Continuous Batching有什么区别]]"
tags: [llm, runtime, batch, tensor]
---

# 模型 Batch 维度与服务端请求 Batch 有什么区别

> [!summary]
> 模型 Batch 维度是一次计算中并列摆放多少条序列；服务端请求 Batch 是 Runtime 本轮选择了哪些请求。二者相关，但不是同一个固定集合。

## 从一个简化形状开始

在 [[00-Embedding向量表示概览|Embedding]] 后，常见输入表示可以写成：

```text
[batch_size, sequence_length, hidden_size]
```

例如 `[3, 8, 5120]` 可以直白理解为：这次计算并列放入 3 条序列，每条当前整理成 8 个位置，每个位置用 5120 个数字表示。

这里第一个 `3` 是张量的 Batch 维度。它描述本次模型调用的数据形状，不负责解释这 3 条序列为何被选中。

## 服务端Batch多了一层选择

服务器可能有 A、B、C、D 四个请求，但本轮资源只允许选择 A、C、D：

```text
等待与运行状态：A B C D
调度器本轮选择：A C D
模型实际看到的Batch维度：3
```

下一轮 B 可能加入，D 可能已经结束。因此服务端 Batch 是动态的请求集合，模型 Batch 维度只是这一次计算的排列结果。

## 长度不同怎么办

不同请求的有效长度可以不同。Runtime 会用位置映射、掩码、分块或紧凑排列等方式告诉执行引擎哪些 Token 属于哪个请求。传统教学常用 Padding 把序列补齐；高性能 Runtime 不一定简单地把所有请求补到相同最大长度。

必需保持的是：序列归属、有效位置和因果关系清楚。具体内存布局属于实现细节。

> [!warning]
> Batch 维度变大不表示单个请求一次知道了多个未来 Token。它通常表示更多独立序列一起推进。

## 一个三请求例子

```text
A：正在预测自己的第21个位置
B：正在预测自己的第7个位置
C：正在处理输入中的一段位置
              ↓
Runtime整理成本轮可执行数据
              ↓
模型分别更新A、B、C，不混合三者语义历史
```

## 理解检查

1. `[3, 8, 5120]` 中的 3 能否直接证明服务器只有三个用户？为什么？
2. 请求长度不同，真正不能丢失的区分信息是什么？

下一篇：[[03-静态Batch与Continuous-Batching有什么区别|静态 Batch 与 Continuous Batching 有什么区别]]。
