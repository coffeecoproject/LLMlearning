---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-输入准备概览|输入准备概览]]"
previous: "[[00-输入准备概览|输入准备概览]]"
next: "[[02-Tokenizer与模型输入张量|Tokenizer与模型输入张量]]"
tags: [llm, inference, prompt, messages, chat-template]
---

# Prompt、Messages 与 Chat Template

> [!summary]
> Messages 是应用层的结构化对话；Chat Template 把角色、内容和边界转换成聊天模型在训练中学过的线性 Token 序列格式。

## 先分清三个对象

| 对象 | 它是什么 | 示例 |
|---|---|---|
| Prompt | 送给模型的整体输入条件 | 指令、问题、背景材料等 |
| Messages | 应用用角色和内容保存的对话结构 | `{role: "user", content: "你好"}` |
| Chat Template | 把 Messages 排成模型协议序列的规则 | 加入 user、assistant、结束标记 |

LLM 最终看到的不是一个天然的“消息列表”。Decoder-only 模型处理的是一条线性 Token 序列，所以角色结构需要被编码进序列。

## 一个教学化例子

应用层消息：

```text
system: 你是一个简洁的助手。
user: 什么是 Token？
```

某种虚构的 Chat Template 可能整理成：

```text
<SYSTEM>你是一个简洁的助手。</SYSTEM>
<USER>什么是 Token？</USER>
<ASSISTANT>
```

最后的 `<ASSISTANT>` 表示接下来应该生成助手内容。真实模型会使用自己的控制 Token 和格式，不能把这个虚构格式照搬给所有模型。

## 为什么不同模型可能需要不同模板

聊天模型在后训练中见过特定的角色和边界格式。两个模型即使来自相似基础架构，也可能分别使用不同控制 Token。模板不是为了美观，而是为了让运行输入匹配模型学过的协议。

Hugging Face 官方文档给出的核心事实是：聊天模型仍然只是续写 Token 序列；`role/content` 消息会被转换为带控制 Token 的序列。`Qwen/Qwen3-8B` 官方示例也先调用 `apply_chat_template`，再执行生成。

## Chat Template 属于哪一层

```text
Chat Template：输入协议与软件准备层
Tokenizer：文本/协议序列到 Token ID 的转换层
Transformer：处理输入向量的模型计算层
```

因此，Chat Template 不属于 Attention 或 FFN，也不表示模型内部真的存放着“消息对象”。

## 常见误解

- **“用户输入一句话就直接进入 Embedding。”** 中间通常还需要消息格式化和 Tokenizer。
- **“Chat Template 就是 Tokenizer 算法。”** 模板决定排列格式，Tokenizer 决定怎样切分和编号。
- **“所有 GPT、Qwen、DeepSeek 都能共用同一个模板。”** 不能默认如此，应使用目标模型官方配置。
- **“模板越复杂，模型越聪明。”** 模板首先要匹配训练协议，复杂本身不等于更好。

## 开放模型观察

`Qwen/Qwen3-8B` 官方示例支持两种连续路径：

```text
apply_chat_template(tokenize=True)
→ 直接得到模型输入
```

或：

```text
apply_chat_template(tokenize=False)
→ 得到格式化文本
→ 再调用配套 Tokenizer
```

软件可以把步骤合并，但“模板整理”和“Token 编码”仍是两个概念。

来源：[Hugging Face Chat Templates 官方文档](https://huggingface.co/docs/transformers/chat_templating)、[Qwen3-8B 官方模型页](https://huggingface.co/Qwen/Qwen3-8B)，核对日期：2026-07-27。

## 理解检查

1. Messages 为什么不能原样作为 Transformer 的输入对象？
2. Chat Template 和 Tokenizer 各自决定什么？
3. 为什么使用错误模板会影响回答？

下一篇：[[02-Tokenizer与模型输入张量|Tokenizer 与模型输入张量]]。
