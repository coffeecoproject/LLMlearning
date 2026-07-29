---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-基础模型是什么概览|基础模型是什么概览]]"
previous: "[[02-Base-Model已经具备什么又缺少什么|Base Model 已经具备什么，又缺少什么]]"
next: "[[04-为什么Base-Model是后训练与适配的起点|为什么 Base Model 是后训练与适配的起点]]"
tags: [llm, base-model, instruct-model, chat-model, post-training]
---

# Base 与 Chat / Instruct Model 有什么区别

> [!summary] 一句话理解
> Chat / Instruct Model 通常是在 Base Model 的基础上继续后训练，使参数更倾向于识别用户要求、生成助手回答，并遵守指定格式与行为偏好。

## 两者不是不同的基本计算原理

在同一模型家族中，Base 与 Instruct 版本经常使用相同或相近的：

- Tokenizer；
- Transformer 架构；
- 层数、Hidden Size 和 Attention 结构；
- Decoder-only 自回归生成方式。

普通运行时，两者通常都执行：

```text
Token IDs
→ Transformer
→ Logits
→ 选择下一个 Token
```

核心差异主要来自后训练的数据、目标和由此改变的参数，而不是 Chat Model 换了一套完全不同的 Transformer。

## Base Model 的默认倾向

```text
输入前缀
→ 继续生成一种合理文本
```

它可能续写文章、问答、代码，也可能模仿 Prompt 的格式。它没有天然义务把每段输入都解释为必须执行的用户命令。

## Chat / Instruct Model 的默认倾向

后训练会给模型大量类似结构：

```text
用户提出要求
→ 助手给出符合要求的回答
```

还可能加入偏好比较、安全行为或任务反馈，使模型更倾向于：

- 将 User 内容识别为请求；
- 以 Assistant 身份回答；
- 遵守格式、语气和约束；
- 在某些条件下拒绝或说明边界；
- 生成工具调用所需的结构。

这些是统计行为倾向，不是百分之百不会失效的硬规则。

## 一个直白对照

Prompt：

```text
请把“苹果是一种水果”翻译成英文，只输出译文。
```

Base Model 可能：

- 给出正确译文；
- 继续写一段新的翻译练习；
- 重复或扩展原问题；
- 输出解释而没有遵守“只输出译文”。

Instruct Model 经过相应训练后，更可能直接输出：

```text
An apple is a fruit.
```

区别不在于 Base 完全不知道英语，而在于 Instruct 版本更稳定地把已有语言能力组织成用户要求的行为。

## Chat Template 起什么作用

Chat Template 会把结构化消息：

```text
System
User
Assistant
```

转换为模型训练时熟悉的特殊 Token 和文本格式。

它解决的是“怎样把角色结构编码进上下文”。但只有模板还不够：如果模型参数没有针对这种格式和行为进行训练，它仍未必稳定扮演助手。

所以：

```text
Chat Template = 输入格式
后训练 = 参数中的行为倾向
```

二者需要匹配，不能互相替代。

## 开放模型对照：Qwen2.5-7B

Qwen 官方分别发布：

| 模型 | 官方训练阶段 |
|---|---|
| `Qwen2.5-7B` | Pretraining |
| `Qwen2.5-7B-Instruct` | Pretraining & Post-training |

两者都属于 Causal Language Model，公开的主要架构参数也对应同一 7B 家族；Instruct 版本另外经过后训练，并提供面向消息的 Chat Template 使用方式。

来源：[Qwen2.5-7B Base 模型卡](https://huggingface.co/Qwen/Qwen2.5-7B)、[Qwen2.5-7B-Instruct 模型卡](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct)，核对日期：2026-07-28。

## 闭源 API 模型的边界

对于 OpenAI 等只通过产品或 API 提供的模型，用户通常接触的是已经经过多阶段训练并接入服务系统的模型产品。

除非官方明确公开，否则不能仅从 API 模型名称推断：

- 它对应哪个内部 Base Checkpoint；
- 具体进行了哪些后训练阶段；
- Base 与产品版本是否一一对应。

开放模型对照用于理解一般阶段，不用于反推闭源模型内部细节。

## 常见误解

### Chat Model 不再做下一 Token 预测

不是。运行时仍然自回归生成，只是参数形成了不同的行为倾向。

### 给 Base Model 套 Chat Template 就等于完成对齐

不是。模板只组织输入，不能代替参数后训练。

### Instruct Model 必定服从所有指令

不是。它仍是概率模型，行为受到训练、上下文、冲突约束和能力边界影响。

## 理解检查

1. Base 与 Instruct 版本的共同基本计算路径是什么？
2. Chat Template 与后训练分别改变什么？
3. 为什么不能用开放模型的阶段直接推断闭源 API 模型？

## 继续学习

- 上一篇：[[02-Base-Model已经具备什么又缺少什么|Base Model 已经具备什么，又缺少什么]]
- 下一篇：[[04-为什么Base-Model是后训练与适配的起点|为什么 Base Model 是后训练与适配的起点]]
