---
type: framework-note
module: 3
status: complete
audience: non-specialist
parent: "[[00-Agent循环与生命周期概览|Agent 循环与生命周期概览]]"
tags: [agent-loop, runtime, framework]
---

# Agent Loop 到底是什么

> [!summary]
> Agent Loop 不是 LLM 内部新增的一种循环结构，而是外部程序在多次模型调用和工具执行之间维持的控制过程。

## 普通模型调用为什么不会自己继续

一次普通调用是：

```text
请求进入模型服务
→ 模型逐 Token 生成
→ 到达停止条件
→ 返回结果
```

模型返回后，这次推理计算就结束了。模型不会在后台自动等待工具、读取数据库或在几分钟后重新唤醒自己。

## Agent 怎样让任务继续

外部 Agent Runtime 读取模型结果。如果模型提出工具调用：

```text
模型调用 1
→ 输出：请求搜索 payment 相关文件

Agent Runtime
→ 校验并执行搜索
→ 获得真实搜索结果

模型调用 2
→ 输入中加入搜索结果
→ 输出：请求读取其中两个文件
```

因此持续性来自：

```text
Runtime 保存状态
+
Runtime 再次调用模型
+
工具和环境返回真实 Observation
```

## Loop 中谁负责什么

| 环节 | 主要责任 |
|---|---|
| 构建上下文 | Context Builder / Agent Runtime |
| 生成候选下一步 | LLM |
| 校验结构和权限 | Agent Runtime |
| 真实执行动作 | Tool Executor / Environment |
| 保存进度 | State Store |
| 判断是否继续 | Agent Runtime 的状态机和策略 |
| 判断目标完成 | 简单系统可由模型自查；高可靠系统由 Verification 与 Acceptance 独立判断 |

## 一次模型回答可以包含很多 Token，为什么还需要多次调用

同一次生成只能基于当前已有上下文向前继续。它无法在输出中凭空获得尚未执行的工具结果。

```text
模型生成“我将运行测试”
≠ 测试已经运行
```

必须先结束或暂停当前模型交互，由外部系统真实运行测试，再把结果放进新的模型调用。

## Agent Loop 不等于无限循环

可靠性要求较高的 Loop 应逐步拥有：

- 明确 Goal；
- 当前状态；
- 允许的动作；
- 最大步骤和预算；
- 等待用户和阻塞状态；
- 错误分类；
- 完成申请，以及与风险相匹配的验证和验收；高风险任务应使用独立验收。

否则它只是一个可能不断消耗 Token 和重复副作用的 `while` 循环。

## 最小因果链

```text
LLM 只能根据当前上下文生成
→ 工具结果必须在模型外产生
→ Agent Runtime 保存并重新组织结果
→ 再次调用模型
→ 多轮调用形成可持续执行
```

下一篇：[[01-一次Run怎样从开始走到结束|一次 Run 怎样从开始走到结束]]。
