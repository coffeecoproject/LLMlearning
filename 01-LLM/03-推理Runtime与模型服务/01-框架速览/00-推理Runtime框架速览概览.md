---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-推理Runtime与模型服务大纲|推理Runtime与模型服务大纲]]"
previous: "[[00-推理Runtime与模型服务大纲|推理Runtime与模型服务大纲]]"
next: "[[00-模型加载与请求边界概览|模型加载与请求边界概览]]"
tags: [llm, inference, runtime, serving, overview, beginner]
---

# 推理 Runtime 框架速览

> [!summary]
> 模型是一组结构和训练后参数；Runtime 是读取这些参数并组织实际计算的软件；模型服务器再为外部应用提供请求、并发、流式传输和管理接口。

## 先分清四个对象

| 对象 | 直白理解 | 典型职责 |
|---|---|---|
| 模型 | 数学结构和训练完成的参数 | 根据输入计算 Hidden States 与 Logits |
| 推理 Runtime | 模型的执行系统 | 加载权重、调用设备、管理 Cache、调度计算 |
| 模型服务器 | 对外提供长期服务的程序 | API、认证、队列、并发、Streaming |
| 应用或 Agent | 使用模型能力的上层系统 | 组织用户体验、任务状态、工具与工作流 |

真实软件可能把 Runtime 和模型服务器打包在一起，但职责仍可区分。

## 一个请求怎样穿过这些层

```text
用户或 Agent 发起请求
→ 模型服务器接收并校验
→ Runtime 准备输入和运行状态
→ 调度 Prefill
→ 调度多轮 Decode
→ 生成结果片段
→ 服务器通过 API / Streaming 返回
```

[[00-单请求推理与生成大纲|单请求推理与生成]]已经解释中间的 Prefill—Decode 因果链；Runtime 研究的是谁来执行、何时执行、和哪些请求一起执行。

## 为什么单用户也会受到 Runtime 影响

即使只看一条回答，体验仍会受到：

- 模型是否已经加载；
- 请求是否排队；
- 当前设备是否繁忙；
- KV Cache 是否有足够内存；
- 输出是否启用 Streaming；
- Runtime 使用什么精度和并行方式。

但这些因素主要改变延迟、吞吐量和可用性，不会把 Runtime 变成模型的长期知识来源。

## 框架层只记五点

1. 模型权重不会自己监听网络端口。
2. Runtime 把模型结构和硬件连接起来。
3. 模型服务器把 Runtime 和外部请求连接起来。
4. 多用户优化不能改变自回归 Token 的逻辑依赖。
5. 客户端能观察 API 行为，不等于能观察闭源服务端内部实现。

继续阅读：[[00-模型加载与请求边界概览|模型加载与请求边界]]。
