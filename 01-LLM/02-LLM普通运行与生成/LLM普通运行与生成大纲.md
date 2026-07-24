---
type: section-outline
module: 1
section: 2
status: active
audience: non-specialist
parent: "[[01-LLM/LLM 模块大纲]]"
tags: [llm, inference, outline]
---

# LLM 普通运行与生成大纲

> 本部分只研究已经训练完成、参数保持固定的模型怎样处理一次输入并逐 Token 生成输出。

```text
文本 → Token ID → 初始表示 → Transformer 前向计算
→ Logits → 选择下一个 Token → 追加并继续生成
```

## 结构

1. [[2.1-普通运行的基本边界概览|普通运行的基本边界]]
2. 一次普通运行的完整流程
3. 输入整理与 Attention Mask
4. Prefill 与 Decode
5. Logits、采样与停止条件
6. 上下文窗口与多轮交互
7. Runtime 服务层：调度、Batch、KV Cache 与资源管理

## 边界

- 前六节先以单个请求理解模型运行。
- Runtime 服务层最后单独学习，不插入模型内部主线。
- 本部分没有训练 Loss、反向传播或 Optimizer 更新。
- CoT 的推理能力不在这里展开。
