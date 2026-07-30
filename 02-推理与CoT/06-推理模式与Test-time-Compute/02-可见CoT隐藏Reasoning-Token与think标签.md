---
type: concept-note
module: 2
status: complete
audience: non-specialist
phase: 运行阶段
parent: "[[00-推理模式与测试时计算概览|推理模式与 Test-time Compute 概览]]"
tags: [cot, reasoning-token, think-tag, visibility]
---

# 可见 CoT、隐藏 Reasoning Token 与 think 标签

> [!summary]
> CoT 是以 Token 序列表达的中间推理步骤；Reasoning Token 是产品或模型接口对推理阶段 Token 的称呼；`<think>` 是部分模型组织这段内容的格式。三者有关联，但不是同一个概念。

## 先分清三层

```text
第一层：神经网络内部计算
Transformer 用 Hidden State、Attention、FFN 等处理上下文

第二层：生成的中间推理 Token
模型把部分中间步骤写成可继续读取的 Token 序列

第三层：用户界面展示
产品决定显示原始过程、推理摘要，还是只显示最终答案
```

最重要的边界是：第二层仍然只是模型内部计算的一种外显文本结果，并不等于第一层的全部真实机制。

## CoT 是什么

Chain of Thought（CoT，思维链）是由一连串中间推理步骤构成的文本路径。

例如：

```text
17 × 24
→ 17 × 20 = 340
→ 17 × 4 = 68
→ 340 + 68 = 408
```

当这些步骤成为已生成 Token 后，后续 Token 可以通过上下文读取它们。这就是 CoT 可能帮助多步任务的直接机制之一。

但 CoT 可能包含遗漏、事后解释或错误步骤，所以“写得像推理”不等于“真实且正确地反映答案形成原因”。

## Reasoning Token 是什么

Reasoning Token 通常指模型在推理阶段使用的 Token。对于支持推理接口的产品，它们可能：

- 计入上下文或输出相关预算；
- 影响延迟和费用；
- 被模型用于形成最终回答；
- 不以原始文字形式展示给用户。

因此可以出现：

```text
模型使用了 Reasoning Token
≠ 用户看到了完整 CoT
```

不同提供方对 Token 统计、保留和展示方式可能不同，不能把一个 API 的定义直接套到所有模型。

## `<think>...</think>` 是什么

它可以理解为一种“分段标记”：

```text
<think>
推理阶段内容
</think>
最终回答
```

部分开放模型会通过训练数据、Chat Template 或生成约定，学会使用这种格式。系统随后可以把 `<think>` 段和最终回答分开处理。

它不是：

- Transformer 必须具备的网络层；
- 所有推理模型统一使用的 Special Token；
- 只要手工写入就必然开启可靠推理的万能指令。

例如，Qwen3 官方资料明确区分 Thinking 与 Non-Thinking 模式，并在具体模板中处理 `<think>` 内容；DeepSeek-R1 的官方说明也给过与 `<think>` 起始格式有关的版本性建议。这些是特定模型的公开约定，不代表所有闭源模型使用相同实现。

## “隐藏”到底隐藏在哪里

需要区分两种说法：

### 对最终用户隐藏

模型服务或应用能处理推理内容，但界面只显示最终答案或摘要。

### 没有对调用者提供原始内容

某些闭源 API 不返回原始内部 CoT，只提供最终回答、Token 用量或经过处理的 reasoning summary。此时上层应用不能假装自己拿到了完整原始推理轨迹。

OpenAI 的公开文档即区分原始推理与 reasoning summary；开放权重模型则可能给部署者更多原始输出控制，但也应避免把未经处理的内部推理直接当作用户答案。

## 搜索是否必须输出可见 CoT

不一定。

完整答案采样可以只让三条路径产生三个最终答案，再用规则比较；这种搜索不要求向用户展示 CoT。

但 Tree of Thoughts 一类按中间状态扩展的搜索，需要控制器取得某种可保存、可评价的中间状态。这个状态可以是简短计划、结构化候选或模型输出片段，也不必原样显示给最终用户。

## 常见误解

### 误解一：CoT 就是模型脑内的全部思考

不对。CoT 是生成出来的 Token 序列，内部还存在大量并不以文字形式呈现的向量计算。

### 误解二：看不见 CoT 就没有推理 Token

不一定。模型或服务可能在最终回答前使用推理 Token，但不公开原文。

### 误解三：看到 `<think>` 就证明推理可靠

不对。标签只划分格式，标签内部仍可能包含错误。

## 开放模型观察

> [!source]
> - [Qwen3 官方仓库](https://github.com/QwenLM/Qwen3)：公开描述 Thinking/Non-Thinking 模式及其模板行为。
> - [DeepSeek-R1 官方仓库](https://github.com/deepseek-ai/DeepSeek-R1)：记录了该系列特定版本的推理格式建议。
> - [OpenAI Reasoning summaries](https://developers.openai.com/cookbook/examples/responses_api/reasoning_items#reasoning-summaries)：说明原始 CoT 不直接暴露、可请求推理摘要的接口边界。
> - [OpenAI gpt-oss Raw CoT 指南](https://developers.openai.com/cookbook/articles/gpt-oss/handle-raw-cot#responses-api)：说明开放权重场景下原始 CoT 与面向用户展示之间仍应分层处理。
> - 核对日期：2026-07-29。格式和接口属于版本敏感信息。

## 理解检查

1. 为什么 CoT 不等于 Transformer 的全部内部计算？
2. 为什么 Reasoning Token 可以存在但不展示？
3. `<think>` 为什么只是特定模型的格式约定，而不是推理的本质？

下一篇：[[03-Reasoning-Effort是什么|Reasoning Effort 是什么]]。
