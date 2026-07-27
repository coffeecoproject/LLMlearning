---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-停止与文本恢复概览|停止与文本恢复概览]]"
previous: "[[01-生成循环怎样停止|生成循环怎样停止]]"
next: "[[00-完整生成循环串联复习概览|完整生成循环串联复习概览]]"
tags: [llm, generation, tokenizer, decode, text]
---

# Token 怎样恢复成文本

> [!summary]
> 生成循环得到的是 Token ID 序列；Tokenizer Decode 按同一套词表和字节规则把它们还原成文本，并按配置处理特殊 Token 和空格等细节。

## 反向路径

```text
输入时：文本 → Tokenizer Encode → Token IDs
输出时：Token IDs → Tokenizer Decode → 文本
```

这里的 Encode/Decode 是 Tokenizer 的转换方向，与 Transformer 的 Encoder-only、Decoder-only 架构不是同一个分类。

## 为什么不能把每个 Token 单独当作一个完整字符

Token 可能表示：

- 一个完整中文字符或词片段；
- 英文词的一部分；
- 空格和标点组合；
- UTF-8 字节的一部分；
- 特殊控制标记。

因此，一个 Token 不一定能独立显示成稳定字符。增量解码器有时需要等待更多 Token 或字节，才能输出合法文本。

## 一个教学化例子

假设两个 Token 分别包含一个 Emoji 的部分字节：

```text
Token A → 不完整字节
Token B → 补齐后组成 😊
```

单独解码 A 可能无法显示，A+B 才能恢复 Emoji。真实切分取决于 Tokenizer，这里只说明为什么“一个 Token = 一个可见字符”不成立。

## 特殊 Token 怎样处理

EOS、角色边界等特殊 Token 参与模型协议，但用户界面通常不希望直接展示它们。Tokenizer Decode 常提供跳过特殊 Token 的选项；工具调用或结构化协议则可能需要在解析层保留并解释某些标记。

## 理解检查

1. Tokenizer Decode 和模型 Decode 阶段分别做什么？
2. 为什么单个 Token 不一定能立刻显示？
3. EOS 为什么常在输出文本中看不到？

下一部分：[[00-完整生成循环串联复习概览|完整生成循环串联复习]]。
