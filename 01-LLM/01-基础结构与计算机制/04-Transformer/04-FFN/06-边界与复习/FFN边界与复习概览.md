---
type: review
module: 1
status: complete
audience: non-specialist
parent: "[[FFN概览]]"
previous: "[[FFN真实模型观察概览]]"
next: "[[FFN与模型知识的关系]]"
tags: [llm, ffn, boundary, review]
---

# FFN 边界与复习

> [!summary]
> FFN 是 Block 内的逐位置参数化变换，不是 Attention、Output Layer、KV Cache、外部知识库或 Agent 路由器。

## 训练与运行边界

> [!info] 两阶段共同
> Dense FFN、门控 FFN 和 MoE 的前向路径在训练与普通运行时都会执行。

> [!info] LLM 训练阶段
> 根据 Loss 和梯度更新 FFN、Gate、Router 和 Expert 参数。MoE 还要处理负载平衡、Expert 容量和训练并行等问题。

> [!info] LLM 运行阶段
> 参数固定，但 FFN 仍需计算；MoE Router 仍要逐 Token 选择 Expert。量化、算子融合、设备通信和 Batch 调度属于运行实现。

一次普通对话不会因为经过 FFN，就自动永久修改模型参数。持续学习需要额外训练或明确的外部记忆系统。

## 模型结构与部署系统的边界

```text
模型结构规定
→ FFN 宽度、Gate、Expert、Router、Top-k 和参数

推理运行时负责
→ 权重怎样加载、放在哪些设备、怎样并行和通信

Agent 负责
→ 目标、上下文、工具和执行循环
```

因此 Agent 使用模型，不等于 Agent 自己实现 FFN 或 MoE Router。

## 必须分清的相邻概念

| 容易混淆 | 准确区别 |
|---|---|
| FFN 与 Attention | 一个逐位置变换，一个跨位置混合 |
| FFN 与 Output Layer | 一个输出 Hidden State，一个输出词表 Logits |
| SwiGLU Gate 与 MoE Router | 一个调节中间维度，一个选择 Expert |
| MoE Top-k 与生成 Top-k | 一个选 Expert，一个选候选 Token |
| Active Parameters 与 Activation Function | 一个是使用的参数集合，一个是非线性函数 |
| 参数化知识与外部实时信息 | 一个来自训练权重，一个来自搜索/文件/API |

## 常见误解检查

- FFN 不直接读取其他 Token，但输入已经上下文化。
- FFN 输出不是最终 Token。
- 升维不是增加 Token 数量。
- 同一层各位置共享参数，不代表所有 Block 共享。
- FFN 常包含大量参数，但不是模型唯一知识存储位置。
- Expert 通常只是 FFN 路径，不是完整 LLM。
- MoE 总参数大，不表示每 Token 执行全部 Expert。
- KV Cache 服务 Attention，不是 FFN 记忆。

## 完整口述题

尝试不用看笔记回答：

1. FFN 位于常见 Transformer Block 的哪里？
2. Attention 与 FFN 各自解决什么问题？
3. 为什么 FFN 能逐位置处理，却仍利用上下文？
4. `hidden_size → intermediate_size → hidden_size` 分别发生什么？
5. 为什么需要 Activation 或 Gate？
6. FFN 主要参数来自哪些投影？
7. Dense FFN 与 MoE 的差别是什么？
8. 为什么总参数不等于每 Token 激活参数？
9. FFN 与参数化知识有什么关系，又为什么不是外部数据库？
10. 训练与运行时，FFN 参数分别处于什么状态？

如果 1—6 能独立回答，基础主线已经掌握；7—10 属于扩展与边界。

下一篇扩展：[[FFN与模型知识的关系|FFN 与模型知识的关系]]。
