---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Block堆叠与Hidden-State流动概览]]"
previous: "[[Final-Hidden-State怎样交给输出层]]"
next: "[[num-hidden-layers与每层参数]]"
tags: [llm, transformer-block, parameters, hidden-state, optional]
---

# Block 参数与边界

> [!summary]
> 这一层解释 Block 数量怎样出现在配置中、不同 Block 的参数通常怎样组织，以及 Hidden States 为什么是临时激活而不是永久模型记忆。

## 阅读顺序

1. [[num-hidden-layers与每层参数|num_hidden_layers 与每层参数]]；
2. [[Hidden-State在训练与运行中的边界|Hidden State 在训练与运行中的边界]]；
3. [[怎样理解不同层的功能|怎样理解不同层的功能]]。

## 这一层回答什么

```text
num_hidden_layers
→ 模型主体堆叠多少个 Transformer Blocks

每层参数
→ Attention、FFN、Norm 等参数通常怎样归属于各 Block

Hidden States
→ 为什么随输入和层次变化，却不自动成为长期参数
```

流水线并行、张量并行、激活检查点和 KV Cache 内存优化只用于说明边界，不在这里展开实现。

下一篇：[[num-hidden-layers与每层参数|num_hidden_layers 与每层参数]]。
