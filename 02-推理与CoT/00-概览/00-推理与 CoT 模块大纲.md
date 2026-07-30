---
type: module-outline
module: 2
status: active
audience: non-specialist
previous: "[[00-LLM 模块大纲|LLM 模块大纲]]"
next: "[[03-Agent/00-概览/00-Agent模块大纲|Agent 模块大纲]]"
tags: [reasoning, cot, inference, search, verification]
---

# 推理与 CoT 模块大纲

## 模块定位

> [!summary]
> 本模块研究语言模型怎样从“直接生成答案”扩展到“生成中间步骤、探索多个候选并利用验证”，以及这些行为为什么仍不等于绝对可靠的思考过程。

它接在 LLM 模块之后：

```text
LLM 模块
→ 已理解 Token、Transformer、逐 Token 生成、Runtime 与训练

推理与 CoT 模块
→ 研究怎样组织更多中间计算，提高复杂任务的成功率

Agent 模块
→ 再把推理接入工具、状态和持续执行循环
```

## 当前学习边界

当前从 **运行阶段的推理表现** 开始，不继续展开 LLM 训练细节。

- RLHF、DPO、RLVR 等训练背景保留在 [[00-后训练与行为对齐概览|后训练与行为对齐]]；
- 本模块重点学习 CoT、采样、搜索、验证和 Test-time Compute；
- 真实工具执行、权限、状态管理和长任务闭环留给 Agent 模块。

## 核心问题

> 模型生成一段看似有逻辑的过程，是否意味着它进行了可靠推理？增加更多中间 Token、候选路径和验证，又分别改变了什么？

## 内容结构

### 01 [[00-推理与CoT框架速览概览|框架速览]]

先区分推理、CoT、搜索和验证，并把它们放回自回归生成主线。

### 02 推理是什么

区分三种常见说法：根据行为结果称模型会推理、把外显步骤称为推理过程、研究模型内部形成答案的计算机制。

### 03 [[02-推理与CoT/03-Chain-of-Thought中间推理文本/00-CoT中间推理文本概览|Chain of Thought 中间推理文本]]

理解 CoT 怎样成为后续生成可继续读取的“文本工作区”，以及它与 Hidden State、最终答案和内部计算的区别。

### 04 推理提示与问题分解

学习 Few-shot CoT、Zero-shot CoT、分解、计划和 Scratchpad；重点理解 Prompt 能诱发什么，不能凭空创造什么。

### 05 [[02-推理与CoT/05-采样搜索与验证/00-采样搜索与验证概览|采样、搜索与验证]]

学习 Self-Consistency、候选重排、Tree of Thoughts、回溯和 Verifier，区分一条生成路径与外部多路径控制。

### 06 [[02-推理与CoT/06-推理模式与Test-time-Compute/00-推理模式与测试时计算概览|推理模式与 Test-time Compute]]

理解普通生成与推理模式、Reasoning Token、Reasoning Effort，以及为什么增加测试时计算可能提高成功率但增加成本。

### 07 忠实性与失败模式

研究 CoT 是否真实反映答案形成原因，以及错误传播、事后合理化、过度思考和自我验证失效。

### 08 推理任务与评估

区分数学、代码、逻辑、事实研究和开放规划，理解 Pass@1、Pass@k、可验证正确率与成本边界。

### 09 [[02-推理与CoT/09-边界与复习/00-推理与CoT边界复习|边界与复习]]

串联模型内部计算、外显 CoT、外部搜索、验证器和 Agent，形成最终判断框架。

## 学习路线

### 只看框架

```text
01 推理与 CoT 框架速览概览
→ 03 CoT 中间推理文本概览
→ 05 采样、搜索与验证概览
→ 06 推理模式与 Test-time Compute 概览
→ 09 推理与 CoT 边界复习
```

这条路线只要求阅读五篇概览。读完 `09`，能够复述完整因果链后即可安全停止，不需要打开各专题的机制、参数或实现笔记。

### 理解完整机制

从 `01` 开始，根据各专题概览中的“理解机制”路线进入子笔记。尚未展开的专题会在后续逐步补充，不影响框架路线完成。

### 暂时不需要深入

- 不推导搜索算法公式；
- 不分析强化学习 Loss；
- 不假定闭源模型公开了内部 CoT 或完整训练方案；
- 不把 Agent 多轮工具执行提前混入模型自身推理。

## 本模块反复使用的五层

```text
模型内部计算
→ Transformer 对当前上下文进行 Hidden State 变换

外显中间步骤
→ 模型生成可继续进入上下文的 CoT Token

外部推理控制
→ 推理控制器或上层系统决定怎样采样、搜索和比较候选

执行与资源调度
→ 模型 Runtime 批处理请求、调度生成并管理 KV Cache 等资源

结果验证
→ 规则、工具、人类或评分器检查结果
```

同一个产品可能同时使用这些层，但不能把它们都简称为“模型在思考”。

## 主要来源

> [!source]
> - [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903)，Wei 等，2022。
> - [Self-Consistency Improves Chain of Thought Reasoning](https://arxiv.org/abs/2203.11171)，Wang 等，2022。
> - [Tree of Thoughts](https://arxiv.org/abs/2305.10601)，Yao 等，2023。
> - [Language Models Don't Always Say What They Think](https://arxiv.org/abs/2305.04388)，Turpin 等，2023。
> - 核对日期：2026-07-29。

## 模块完成标准

- 不把“输出了思维链”直接等同于“公开了全部内部机制”；
- 能解释中间 Token 为什么可能提高多步任务表现；
- 能区分单路径生成、多候选采样、搜索与验证；
- 能判断一项推理能力来自模型 Weight、运行时计算还是外部系统；
- 能说明推理表现提高为什么仍不等于事实和任务结果得到保证。
- 能区分一次结果的验证与模型或系统整体能力的评估。
- 能判断什么时候问题已经从模型推理进入 Agent 的状态、工具与控制闭环。
