---
type: review
module: 1
status: complete
audience: non-specialist
parent: "[[完整Transformer-Block串联复习概览]]"
previous: "[[完整Block形状追踪]]"
next: "[[Output-Layer输出层概览]]"
tags: [llm, transformer, review, misconceptions]
---

# Transformer 基础结构最终检查

> [!summary]
> 如果能够不看笔记解释从 Hidden States 进入一个 Block，到 Final Hidden States 交给 Output Layer 的完整因果链，Transformer 基础结构就已经形成闭环。

## 先完成一段口述

尝试用自己的话补全：

```text
Embedding 形成初始 Hidden States；位置影响根据具体方案在输入附近或 Attention 内部进入。

进入一个常见 Pre-Norm Block 后：
先……
再通过 Attention……
然后 Residual……
接着……
FFN……
最后……

多个 Block 堆叠后……
再经过 Final Norm 和 LM Head……
```

## 标准参考路线

```text
Token Embedding / 初始 Hidden States
→ 位置影响按 Absolute Position、RoPE 或 ALiBi 的方案在对应位置进入
→ Norm
→ Causal Self-Attention：跨位置汇集信息
→ Residual：加入 Attention 变化
→ Norm
→ FFN：逐位置加工上下文化表示
→ Residual：加入 FFN 变化
→ 下一 Block
→ Final Hidden States
→ Final Norm
→ LM Head
→ Logits
```

## 十个核心问题

1. Transformer 为什么不能只有初始 Embedding？
2. Attention 为什么要区分 Q、K、V？
3. Causal Mask 与 Position 机制有什么区别？
4. Attention 与 FFN 分别怎样处理位置关系？
5. Residual 为什么需要分支输出回到 `hidden_size`？
6. LayerNorm 与 RMSNorm 的主要差别是什么？
7. Pre-Norm 与 Post-Norm 描述的是什么？
8. `hidden_size`、`intermediate_size`、`num_hidden_layers` 分别表示什么？
9. Hidden States、模型参数和 KV Cache 有什么区别？
10. Final Hidden States 为什么还不是下一个 Token？

## 必须能纠正的误解

### “Transformer 就是 Attention”

Transformer Block 还包含 FFN、Residual 和 Normalization。

### “Decoder-only 不执行编码”

Decoder-only 是 Transformer 架构类型，不会跳过 Tokenizer 编码和 Embedding。

### “Attention Weight 就是完整思考过程”

Weight 只描述特定层、特定 Head 中的信息份量；模型行为还依赖 Value、FFN、Residual、多层堆叠和 Output Layer。

### “intermediate_size 会逐层累加”

它是每个 FFN 内部的临时宽度，离开 FFN 前会压回 `hidden_size`。

### “Hidden State 变化表示模型在运行时学习了新参数”

Hidden State 是当前输入的临时激活；普通运行时模型参数保持固定。

### “Final Hidden State 已经是概率”

还需要 Final Norm、LM Head，并根据阶段进入 Loss 或生成选择流程。

## 阶段总边界

```text
本专题完成的内容
→ Transformer 的结构和一次前向计算

训练模块继续回答
→ 参数怎样根据 Loss 和梯度更新

运行模块继续回答
→ KV Cache、逐 Token Decode、采样和请求调度

Agent 模块继续回答
→ 模型怎样使用工具、记忆和执行循环
```

## 完成判断

如果十个核心问题中 1—8 能独立回答，已经掌握 Transformer 基础结构；9—10 用于确认没有把运行系统和 Output Layer 提前混入。

Transformer 专题完成。下一专题：[[Output-Layer输出层概览|Output Layer 输出层]]。
