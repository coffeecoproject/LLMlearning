---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-API与流式传输概览|API与流式传输概览]]"
previous: "[[03-请求响应与流式事件分别是什么|请求、响应与流式事件分别是什么]]"
next: "[[05-网络分块为什么不等于模型Token|网络分块为什么不等于模型Token]]"
tags: [llm, generation, streaming, api, user-interface]
---

# Streaming 为什么逐步显示

> [!summary]
> 模型本来就逐 Token 生成；Streaming 让服务端不必等完整回答结束，而是在可解码片段准备好后持续向客户端发送事件或文本增量。

## 没有 Streaming

```text
服务端生成完整回答
→ 一次性返回
→ 用户长时间看不到中间结果
```

## 使用 Streaming

```text
生成若干 Token
→ 恢复出可显示片段
→ 发送增量事件
→ 客户端追加显示
→ 继续生成
```

Streaming 主要改善**感知等待体验**。它不必然缩短生成完整回答所需的总计算时间。

## 为什么界面片段不一定等于一个 Token

服务端可能：

- 等待字节组成合法文字；
- 合并多个 Token 再发送；
- 把一个较长文本增量拆成网络事件；
- 分别发送文本、工具调用参数、状态和完成事件。

所以“屏幕上一次出现一个字”不能用来反推模型的 Token 划分。

## 模型层和 API 层的边界

逐 Token 条件生成来自模型和生成循环；怎样封装网络事件、何时刷新 UI、断线怎样处理属于服务/API/客户端层。OpenAI Responses API 的官方参考说明，当启用流式模式时，服务端会在生成过程中发送 Server-Sent Events；这能证明接口行为，但不能据此推断闭源模型内部缓存结构。

来源：[OpenAI Responses Streaming 官方参考](https://platform.openai.com/docs/api-reference/responses-streaming)，核对日期：2026-07-27。

## 理解检查

1. Streaming 为什么能更早让用户看到内容？
2. 它为什么不一定减少完整回答的总计算量？
3. 为什么一个流式文本片段不等于一个 Token？

下一篇：[[05-网络分块为什么不等于模型Token|网络分块为什么不等于模型 Token]]。
