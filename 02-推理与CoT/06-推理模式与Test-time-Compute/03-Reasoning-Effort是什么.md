---
type: concept-note
module: 2
status: complete
audience: non-specialist
phase: 运行阶段
parent: "[[00-推理模式与测试时计算概览|推理模式与 Test-time Compute 概览]]"
tags: [reasoning-effort, test-time-compute, reasoning-budget]
---

# Reasoning Effort 是什么

> [!summary]
> Reasoning Effort（推理投入）是一次模型调用的运行阶段控制项，用来引导支持该能力的模型在形成最终回答前投入多少推理计算；它不是统一、精确的 Token 数开关。

## 为什么需要这个控制项

不同问题需要的处理程度不同：

```text
“法国首都是哪里？”
→ 通常不需要很高推理投入

“检查大型项目中的并发扣款问题，并证明修复没有破坏正常支付”
→ 需要更充分的分析、证据和验证
```

如果所有问题都使用最高投入，简单问题会产生不必要的延迟和成本；如果所有问题都使用最低投入，复杂问题可能来不及展开足够的中间处理。

## 它直接控制什么

准确说法是：

```text
Reasoning Effort
→ 引导一次调用投入多少推理阶段计算
```

“推理深度”可以帮助直观理解，但它不是一个所有模型都用相同刻度测量的严格物理量。

不同模型和服务可能把这个控制映射到不同内部策略，例如允许更多推理 Token、采用不同终止倾向或按难度自适应分配预算。闭源服务若未公开细节，就只能确认接口行为，不能反推其完整内部算法。

## 它不是什么

| 容易混淆的参数 | 实际控制对象 |
|---|---|
| Reasoning Effort | 一条调用的推理投入 |
| Search Width | 建立多少条候选路径 |
| Temperature | 从 Token 概率分布中采样时的随机程度 |
| Max Output Tokens | 最多允许产生多少输出 Token |
| Context Window | 一次请求最多能容纳多少上下文 Token |
| Verbosity | 面向用户的答案写得多详细 |

因此：

```text
high
≠ 自动把同一问题发送三次
≠ 自动提高 Temperature
≠ 要求显示更长答案
≠ 扩大 Context Window
```

## 与 Reasoning Token 的关系

推理投入提高时，模型通常有机会使用更多 Reasoning Token，但不能把每个档位理解为固定数字：

```text
low = 恰好 500 个 Token
medium = 恰好 2,000 个 Token
high = 恰好 8,000 个 Token
```

这种理解通常不成立。实际使用量会受到问题难度、模型版本、服务策略、输出限制和提前完成等因素影响。

## 一次调用和多轮 Agent 的区别

Reasoning Effort 主要描述一条模型调用内部的推理投入。

Agent 完成大型任务时还可能进行：

```text
模型调用 1：决定读取哪些文件
→ 工具返回真实文件
模型调用 2：提出根因
→ 运行测试
模型调用 3：根据失败继续修正
```

这属于外部执行循环。每一轮调用都可以各自设置 Reasoning Effort，但多轮循环本身不是 Reasoning Effort 创造的。

## 常见误解

### 误解一：推理投入越高，答案必然越对

不对。高投入只是提供更多计算机会，模型仍可能误解问题、缺少事实或在错误方向上过度思考。

### 误解二：高投入一定产生更长的可见 CoT

不对。原始推理过程可能不展示，最终回答也可能很短。

### 误解三：所有模型都有相同档位

不对。可用值、默认值和实际行为都与模型版本及服务接口有关。

## 版本敏感事实

> [!source]
> OpenAI 当前官方指南将 `reasoning.effort` 描述为“引导模型投入多少推理计算”的参数，并明确支持的档位依模型而异。指南列出的可能值包括 `none`、`minimal`、`low`、`medium`、`high`、`xhigh` 和 `max`，但不代表每个模型都支持全部档位。
>
> 来源：[OpenAI Reasoning models guide](https://developers.openai.com/api/docs/guides/reasoning#reasoning-effort)。核对日期：2026-07-29。

## 理解检查

1. 为什么 Reasoning Effort 不是固定 Token 数？
2. 它与 Temperature、回答长度分别有什么区别？
3. 为什么一次高投入调用不等于一个多轮 Agent？

下一篇：[[04-low-medium-high控制了什么|low、medium、high 控制了什么]]。
