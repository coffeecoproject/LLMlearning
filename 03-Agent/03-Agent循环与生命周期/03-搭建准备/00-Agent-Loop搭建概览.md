---
type: implementation-overview
module: 3
learning_layer: cross-layer
status: complete
audience: non-specialist
parent: "[[00-Agent循环与生命周期概览|Agent 循环与生命周期概览]]"
tags: [agent-loop, implementation, controller]
---

# Agent Loop 搭建概览

> [!summary]
> 第一版 Agent Loop 只需把“模型决策—工具执行—结果回传—继续或停止”跑通；状态机、持久恢复、事件协议与独立验收是按风险增加的控制能力。

> [!info] 层级定位
> 第一版只需模型调用、一个工具、Observation、最大步数和停止条件；事件协议、恢复、幂等与独立完成边界按风险继续增加。

## Agent 基础结构：最小组件

```text
Simple Loop
├── Input Assembler
├── Model Client
├── Response Branch（最终回答或 Tool Call）
├── Tool Dispatcher
└── Step Limit / Stop Condition
```

这些职责可以只是同一程序中的几个函数，不要求独立服务、Run 数据库或正式状态机。

## 基础循环的一轮

```text
1. 把当前请求、工具说明和已有结果组装成模型输入
2. 发起一次 Model Call
3. 如果模型给出最终回答，就结束
4. 如果模型请求工具，检查工具名与参数
5. 执行工具并取得 Tool Result
6. 把结果加入下一次输入
7. 未达到最大步数时再次调用模型
```

## 第一版建议能力

- 直接把本次用户请求作为当前 Goal；
- 一个只读工具；
- 能区分最终回答与 Tool Call；
- 对工具名和参数做程序校验；
- 一个最大步数与明确停止条件；
- 保留当前 Turn 中必要的消息和 Tool Result。

## 持续与高可靠理想结构（选读）

当任务需要跨 Turn、进程重启、外部副作用或独立验收时，再扩展为：

```text
Run Controller
├── State Reader / Writer
├── Context Builder
├── Model Client / Response Decoder
├── Tool Dispatcher
├── Transition Policy
├── Budget Manager
└── Event / Audit Writer
```

此时一轮可能变成：加载 Run 与版本 → 检查状态和预算 → 构建 Context Package → 校验事件与权限 → 执行动作 → 原子保存 Observation 和状态 → 安排下一轮。正式 Goal、Run 状态机、Completion Request 和独立 Acceptance 都属于这一层。

## 暂时不要加入

- 多 Agent；
- 动态任意代码执行；
- 无上限自动重试；
- 完全由模型生成状态名称；
- 模型可以修改自身权限；
- 分布式任务队列和复杂图调度。

## 搭建顺序

1. 接通一次模型调用，并能返回最终回答；
2. 接入一个只读工具，把 Tool Result 返回模型；
3. 加入最大步数、错误与停止条件；
4. 为循环写确定性测试；
5. 确有跨 Turn 或恢复需求时，再加入 Run、事件、持久化和状态机；
6. 高风险任务最后增加权限审批、Evidence 与独立 Acceptance。

下一篇：[[01-控制循环与事件协议|控制循环与事件协议]]。
