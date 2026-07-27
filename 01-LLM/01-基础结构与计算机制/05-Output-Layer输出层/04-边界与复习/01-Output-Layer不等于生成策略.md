---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer边界与复习概览|Output-Layer边界与复习概览]]"
previous: "[[00-Output-Layer边界与复习概览|Output-Layer边界与复习概览]]"
next: "[[02-从文本输入到Logits完整串联|从文本输入到Logits完整串联]]"
tags: [llm, output-layer, decoding, sampling, boundary]
---

# Output Layer 不等于生成策略

> [!summary]
> Output Layer 负责给词表候选打分；生成策略负责怎样使用这些分数选出 Token。两者前后相连，但不是同一件事。

## 用职责拆开

### Output Layer

```text
输入：Final Hidden State
动作：Final Norm、LM Head 投影
输出：词表 Logits
```

### 生成策略

```text
输入：Logits
动作：Temperature、屏蔽、Top-k、Top-p、贪心或采样等
输出：被选中的下一个 Token ID
```

### Tokenizer Decode

```text
输入：一个或多个 Token ID
动作：按 Tokenizer 规则还原
输出：字节或可显示文本
```

## 为什么容易混在一起

用户按下发送后，这些环节通常在很短时间内连续完成。最终界面只显示文字，内部的 Hidden State、Logits 和采样步骤都不可见，于是容易形成“Output Layer 直接吐出整段文字”的印象。

真实自回归生成更接近：

```text
模型产生一组 Logits
→ 系统选一个 Token
→ 把这个 Token 加入上下文
→ 再运行下一步
→ 重复直到停止
```

## 同一组 Logits 可以产生不同输出吗

可以。如果使用带随机性的采样策略，同一组概率分布可能抽到不同 Token；改变 Temperature、Top-k 或 Top-p 也会改变候选分布或可选集合。

这不意味着 LM Head 自己随机改变了权重。变化可能来自模型之外或模型输出之后的选择规则。

## Output Layer 也不等于 API 响应

聊天 API 可能还包含：

- 服务端 Prompt 组织；
- 安全或格式约束；
- 工具调用协议；
- 流式输出；
- Token 使用量统计；
- 多请求调度。

这些上层系统使用模型输出，但不属于 LM Head 的内部机制。

## 常见误解

- Softmax、Temperature 与 Top-p 不是三层新的 Transformer Block。
- 选择了 Token ID 后，仍需 Tokenizer Decode 才能显示文字。
- “模型输出概率”不等于 API 一定公开完整概率。
- Agent 的工具选择可能使用模型生成结果，但 Agent 不等于 Output Layer。

## 理解检查

1. LM Head 和采样策略各自输出什么？
2. 为什么相同 Prompt 有时会得到不同回答？
3. Token ID 被选择后，为什么还不能直接当作文字显示？

下一篇：[[02-从文本输入到Logits完整串联|从文本输入到 Logits 完整串联]]。
