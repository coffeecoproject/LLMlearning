---
type: topic-overview
module: 3
learning_layer: cross-layer
status: complete
audience: non-specialist
parent: "[[00-Agent模块大纲|Agent 模块大纲]]"
tags: [agent, framework, loop, tools, state]
---

# Agent 框架速览概览

> [!summary]
> Agent 是一个由外部程序控制的执行系统：LLM 根据本轮上下文提出下一步，Agent Runtime 决定是否执行工具、怎样保存结果、是否继续以及何时接受或停止。

## 先确定学习边界

Agent 没有唯一标准实现。本页先给出所有 Agent 都能映射的基本循环，再展示高可靠系统可能增加的能力。一个组件是否独立存在，和它所承担的职责是否存在，是两个问题。

```text
最小 Agent
用户请求 → 模型 → 工具 → Observation → 再次调用模型 → 停止

持续 Agent
最小 Agent + 持久目标/状态 + 恢复 + 预算

高可靠 Agent
持续 Agent + 任务契约 + 阶段控制 + 独立证据与验收
```

## 为什么只有 LLM 还不够

一次普通模型调用通常是：

```text
输入
→ LLM 生成
→ 输出
```

但“修复一个项目中的重复扣款问题”还需要：

```text
确认目标
→ 搜索代码
→ 读取真实文件
→ 修改代码
→ 运行测试
→ 读取失败结果
→ 再次修正
→ 验证业务条件
→ 保存结果
```

LLM 可以生成分析和动作建议，但文件、测试、数据库和任务状态都存在于模型之外。因此必须有一个程序把多次模型调用和真实环境连接起来。

## Agent 基础结构

~~~text
当前请求或目标
→ 程序构造模型输入
→ LLM 提出回答或 Tool Call
→ 程序执行允许的工具
→ Tool Result 成为下一次输入
→ 继续或停止
~~~

这条链成立，就已经具备 Agent 的基本行动循环。状态存储、Context Builder、Verifier 等职责可以只是同一个程序中的少量逻辑，不要求拆成独立组件。三层结构的判断见 [[01-Agent基础结构与高可靠理想结构怎样区分|Agent 基础结构与高可靠理想结构怎样区分]]。

## 持续与高可靠能力地图（选读）

```text
用户目标
    ↓
Agent Runtime
    ├── 状态存储
    ├── Context Builder
    ├── 工具注册与权限
    ├── 循环与停止规则
    └── 验证与日志
            ↓
        模型服务
            ↓
           LLM
            ↓
提问 / 回答 / 计划 / 工具调用 / 完成申请
            ↓
Agent Runtime 解析并校验
    ├── 按需保存 Plan 或 Task
    ├── 授权并执行 Tool Call
    └── 检查 Completion Request
            ↓
Observation / 产物 / Evidence
            ↓
更新状态，继续下一轮或结束
```

## 一次循环迭代怎样发生

以“找出测试失败原因”为例：

```text
1. Agent 读取当前 Goal 和运行状态；
2. Context Builder 放入错误日志和相关约束；
3. 发起一次 Model Call；
4. LLM 生成：请求搜索 `payment` 相关文件；
5. Agent 检查这个工具是否允许、参数是否合法；
6. 搜索工具真实执行；
7. 搜索结果保存为 Observation；
8. Agent 重新构建上下文并发起下一次 Model Call。
```

模型并不是在后台一直运行。常见 Agent 是由外部循环反复发起模型调用：

```text
模型调用
→ 工具执行
→ 模型调用
→ 工具执行
→ ……
```

## 先区分三组对象

### 任务执行对象

| 名称 | 最小理解 |
|---|---|
| Goal | 用户希望最终实现的目标及完成边界 |
| Run | 围绕一个 Goal 发起的一次受控运行 |
| Phase | Run 中具有不同权限和退出条件的可选阶段 |
| Step | Run 中一个可记录的处理步骤或动作 |
| Attempt | 对某个阶段或任务的一次执行尝试 |

### 产品交互与协议对象

| 名称 | 最小理解 |
|---|---|
| Thread | 一整个持续对话或任务交互容器 |
| Turn | 在 Codex 中，从一次用户输入到最终 Agent Message 的完整处理过程 |
| Item | Turn 中可记录和展示的一条事件 |
| Model Call | Agent Runtime 对模型服务的一次实际请求和响应 |

### 循环反馈对象

| 名称 | 最小理解 |
|---|---|
| Action | Agent 准备执行的动作，例如调用工具 |
| Tool Result | 工具返回的原始结果 |
| Observation | Agent 能够记录并用于下一步决策的环境反馈 |

一个 Codex Turn 可以包含多次 Model Call、Tool Call 和 Observation。上述名称不是所有框架统一使用的标准，详细关系见 [[01-Thread-Turn-Item与Model-Call|Thread、Turn、Item 与 Model Call]]。

## 任务复杂后需要逐步外置的七个问题

| 问题 | 对应系统能力 |
|---|---|
| 要完成什么 | Goal、范围、成功条件 |
| 当前进行到哪里 | Run、Phase、Step、Attempt 和持久状态 |
| 本轮模型应看到什么 | Context Builder / Context Compiler |
| 模型可以建议什么 | 回答、计划、结构化动作或工具调用 |
| 哪些动作真的可以执行 | 工具、权限、Sandbox 和 Approval |
| 失败后怎样继续 | Observation、重试、恢复和重新规划 |
| 谁判断真的完成 | Verifier、Evidence 和 Acceptance |

最小 Agent 不需要把七项都做成独立组件，也可能暂时没有持久 Run、正式 Scope、独立 Evidence 或 Acceptance。任务越长、风险越高，这些问题才越需要从模型临场判断中外置出来。

复杂任务还会增加“怎样把 Goal 分成 Tasks、怎样表示依赖”的规划问题；简单 Agent 可以直接逐轮选择下一步，所以显式 Plan 不是最小 Agent 的必需对象。

## Agent、Workflow 和模型调用

### 普通模型调用

```text
输入基本确定
→ 生成一次结果
→ 返回
```

适合解释、改写、分类和一次性草稿。

### Workflow

```text
程序预先规定步骤 A → B → C
```

适合流程稳定、分支明确、规则能够编码的任务。Workflow 中可以调用 LLM，但步骤控制仍主要来自程序。

### Agent

```text
系统给出目标和边界
→ 模型根据观察动态选择下一步
→ 外部系统执行并反馈
```

适合无法提前写死每一步、需要根据环境结果持续调整的任务。

三者不是非黑即白。实际产品经常是：

```text
确定性 Workflow 作为外框
+
Agent 在受限阶段内动态决策
```

## 模型输出为什么只是候选动作

LLM 可能生成：

```text
调用 delete_file(path="...")
```

这只是模型产生的结构化 Token，不代表文件已经删除，也不代表它有权删除。

正确链路是：

```text
LLM 提出 Tool Call
→ Agent 校验工具、参数、权限和当前阶段
→ 必要时请求用户批准
→ Tool Executor 执行
→ 系统记录真实结果
```

这一边界是 Agent 安全性的基础。

## 状态、上下文和记忆不是同一件事

```text
状态 State
→ 系统当前真实进行到哪里

上下文 Context
→ 本次模型调用实际收到哪些 Token

工作记忆 Working Memory
→ 为当前任务保留、可在未来调用中重新选取的信息

长期学习 Continual Learning
→ 跨任务积累并更新经验，属于后续模块
```

Agent 不应只依赖越来越长的聊天记录维持状态。可靠系统通常把关键状态保存在模型上下文之外，再按每一轮需要重新编译进入上下文。

## 高可靠任务为什么不能只听模型声明

模型说“已经完成”只能视为一个 Completion Request。

```text
模型声称完成
→ 系统检查实际产物
→ 运行规定的验证
→ 形成 Evidence
→ Acceptance 根据成功条件判断
→ 接受、要求修复或阻塞
```

如果模型既执行任务又独自定义检查范围、解释结果并批准自己，就容易产生“看起来完成”的假闭环。

## Agent 的核心边界

> [!warning]
> Agent 增加了行动能力，也同时扩大了错误的影响范围。模型的不确定性不会因为接入工具而消失，只会从文字错误扩展为真实副作用。

因此 Agent 需要把不确定性限制在：

- 有界目标；
- 有限步骤和预算；
- 最小权限；
- 可恢复状态；
- 可验证产物；
- 明确的人工决策点。

## 阅读路线

### 只看框架

读完本页后进入 [[00-目标与任务契约概览|目标与任务契约概览]]，再按照“Agent Loop → 运行状态 → 上下文 → 模型决策 → 工具 → Observation → 验收”的主线阅读。目标专题会区分直接使用用户请求、持久化 Thread Goal 和完整任务契约三种层级；Goal Intake 只作为高可靠可选扩展，不属于主线必读内容。

### 理解机制

重点学习 Agent Loop、运行状态、Context Builder、模型决策、Tool Call 和 Observation 反馈。规划与任务图可以根据任务复杂度选读。

### 为搭建做准备

继续学习验证、权限、Agent Runtime、可观测性和搭建路线，并在单 Agent 闭环可靠后再进入多 Agent。

## 框架停止点

如果你能解释“LLM 根据本次 Context 生成候选下一步，外部程序执行工具并把结果送回下一次调用”，就已经建立 Agent 的基础结构。持久状态、恢复、权限分层和独立验收属于后续增强。
