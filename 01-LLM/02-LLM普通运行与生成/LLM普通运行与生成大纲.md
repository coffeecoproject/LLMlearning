---
type: section-outline
module: 1
status: active
audience: non-specialist
parent: "[[01-LLM/LLM 模块大纲]]"
tags: [llm, inference, outline]
---

# LLM 普通运行与生成大纲

> 本部分只研究已经训练完成、参数保持固定的模型怎样处理一次输入并逐 Token 生成输出。

```text
用户消息
→ Chat Template
→ Tokenizer 与输入张量
→ Prefill 并建立 KV Cache
→ 读取最后有效位置的 Logits
→ Logits 处理与 Token 选择
→ 追加新 Token
→ 使用 KV Cache 继续 Decode
→ 停止并恢复为文本
```

## 结构

1. [[普通运行的基本边界概览|普通运行的基本边界]]：固定参数不等于没有运行状态；
2. Prompt、Messages 与 Chat Template：用户看到的消息怎样变成模型协议文本；
3. Tokenizer 与输入张量：`input_ids`、有效位置 Mask 与 Position 信息分别是什么；
4. 单请求完整生命周期：从接收输入到输出结束；
5. Prefill、KV Cache 与 Decode：一次处理输入和逐步续写怎样分工；
6. Logits Processing、Temperature 与选择策略：Greedy、Sampling、Top-k、Top-p；
7. 停止条件、Tokenizer Decode 与 Streaming：Token 怎样变回用户看到的文字；
8. 上下文窗口、Truncation 与多轮交互：模型没有自动永久记忆；
9. 单请求效率：输入长度、输出长度、首 Token 延迟（TTFT）、逐 Token 延迟、显存与计算量；
10. Runtime 服务层：Batch、连续批处理（Continuous Batching）、调度、分页式 KV Cache 管理、并行与部署形态。

## 边界

- 前九节先以单个请求理解模型运行。
- KV Cache 本身属于单请求增量生成；分页式 KV Cache、跨请求复用与内存调度属于 Runtime 服务层。
- Runtime 服务层最后单独学习，不插入单请求模型主线。
- 本部分没有训练 Loss、反向传播或 Optimizer 更新。
- CoT 的推理能力不在这里展开。

## 真实实现校验

Qwen3-8B 官方使用示例先对消息执行 `apply_chat_template`，再 Tokenize 并调用生成；DeepSeek-V3 官方仓库则把单模型推理与 SGLang、LMDeploy、TensorRT-LLM 等服务框架分开列出。两者共同说明“模型运行链”和“服务系统链”应该先分开，再建立联系。

来源：[Qwen3-8B 官方模型页](https://huggingface.co/Qwen/Qwen3-8B)、[DeepSeek-V3 官方仓库](https://github.com/deepseek-ai/DeepSeek-V3)，核对日期：2026-07-27。
