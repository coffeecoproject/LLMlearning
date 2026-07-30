---
type: topic-overview
module: 3
status: planned
audience: non-specialist
parent: "[[00-Agent模块大纲|Agent 模块大纲]]"
tags: [agent-runtime, service, app-server, queue, concurrency]
---

# Agent Runtime 与服务架构概览

> [!summary]
> Agent Runtime 是负责目标运行、状态、模型调用、工具、权限和恢复的控制程序；它位于业务界面与模型服务之间，不等于执行 Transformer 的模型 Runtime。

## 两种 Runtime

```text
Agent Runtime
→ 管理 Goal、Run、Turn、Tool、State、Policy 和 Recovery

模型 Runtime
→ 加载模型并执行 Prefill、Decode、Batch 和 KV Cache
```

二者可能由不同公司、进程或服务器提供。

## 通用部署路径

```text
GUI / CLI / API
→ Agent App Server 或本地 Agent Runtime
→ 模型 API / 自建模型服务
→ 模型 Runtime
→ LLM

Agent Runtime
↔ 状态数据库、任务队列、工具执行器、Verifier、日志系统
```

## 本专题要解决的问题

- 本地 Agent、服务器 Agent 和混合架构怎样选择；
- App Server、Agent Runtime、模型服务和 vLLM 各在哪一层；
- 多个 Run 如何排队、并发和隔离；
- 长任务怎样处理进程重启、超时和后台执行；
- Thread/Turn 协议和流式事件怎样映射到 Agent 状态；
- 模型供应商切换怎样保持控制层稳定；
- Agent 服务与业务后端怎样划分。

## 关键边界

Agent Runtime 可以选择模型和推理投入、构建请求并组织循环，但不会因此获得模型 Weight 内部能力，也不会替代模型 Runtime 的计算职责。

## 后续展开

1. 本地、服务端与混合部署；
2. App Server、模型 API 与适配器；
3. Run 调度、队列和并发；
4. 长任务、后台执行和恢复；
5. 流式事件和客户端状态；
6. Provider Adapter 与协议版本。

## 阅读路线

- 只看框架：记住两种 Runtime 的区别；
- 理解机制：画出一次请求跨进程的完整路径；
- 为搭建做准备：先做单进程 Runtime，再根据并发和可靠性需求拆服务。
