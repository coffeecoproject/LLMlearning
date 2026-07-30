---
type: implementation-overview
module: 3
status: complete
audience: non-specialist
parent: "[[00-Agent循环与生命周期概览|Agent 循环与生命周期概览]]"
tags: [agent-loop, implementation, controller]
---

# Agent Loop 搭建概览

> [!summary]
> 搭建 Agent Loop 的重点不是写出能反复调用模型的循环，而是定义每轮输入、允许结果、状态转换、预算、错误和停止边界。

## 最小组件

```text
Run Controller
├── State Reader / Writer
├── Context Builder
├── Model Client
├── Response Decoder
├── Tool Dispatcher
├── Transition Policy
├── Budget Manager
└── Event / Audit Writer
```

这些组件第一版可以位于同一个进程，但职责需要可区分和测试。

## 一轮控制顺序

```text
1. 加载当前 Run 和版本
2. 判断当前状态是否可执行
3. 检查取消、预算和超时
4. 构建 Context Package
5. 发起一次模型 Turn
6. 严格解析返回事件
7. 校验权限和当前状态
8. 执行最多一个明确的下一动作或受控动作组
9. 保存 Observation、结果和状态变化
10. 决定是否安排下一轮
```

“一次只提交一个明确状态变化”通常比让 CLI 或模型连续改多个权威对象更容易恢复和审计。

## 第一版建议能力

- 一个正式 Goal；
- 一个 Run；
- `READY / RUNNING / WAITING / VERIFYING / COMPLETED / FAILED / CANCELLED` 一组说明性状态；
- 一个只读工具；
- 一个最大 Turn 数；
- 一个 Completion Request；
- 一个确定性验证条件；
- 所有 Turn 和 Tool Call 可追踪。

## 暂时不要加入

- 多 Agent；
- 动态任意代码执行；
- 无上限自动重试；
- 完全由模型生成状态名称；
- 模型可以修改自身权限；
- 分布式任务队列和复杂图调度。

## 搭建顺序

1. 先实现没有 LLM 的确定性状态机测试；
2. 接入返回固定事件的 Fake Model；
3. 接入一个只读工具；
4. 再替换成真实模型；
5. 加入错误、取消和预算；
6. 最后连接持久化和恢复。

下一篇：[[01-控制循环与事件协议|控制循环与事件协议]]。
